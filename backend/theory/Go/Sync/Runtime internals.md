- `runtime_Semacquire(addr)` / `runtime_Semrelease(addr)` — **внутренний семафор** Go runtime. Фундамент **всех** sync-примитивов: Mutex, RWMutex, WaitGroup, Once ^rsem-def
- `gopark` / `goready` — **парковка и пробуждение** горутин. Более низкий уровень: каналы, таймеры, netpoll, select — всё через gopark ^gopark-def
- `procPin` / `procUnpin` — **привязать горутину к P** (не даёт планировщику перебросить на другой P). Используется в sync.Pool ^procpin-def

---

## runtime_Semacquire / runtime_Semrelease

Внутренний семафор. Фундамент Mutex, RWMutex, WaitGroup, Once. ^rsem-mechanism

```
runtime_Semacquire(&s):
  1. if *s > 0 → *s--; return (fast path, без блокировки)
  2. else → создать sudog → положить в treap-очередь по addr → gopark (засыпаем)

runtime_Semrelease(&s):
  1. *s++
  2. если есть ожидающие в очереди по addr → goready (будим одну горутину)
```

^rsem-flow

**sudog** — структура "спящей горутины", содержит указатель на G, канал пробуждения. Переиспользуется через пул. ^rsem-sudog

### Где используется

```go
// sync.Mutex.Lock() упрощённо:
func (m *Mutex) lockSlow() {
    // ... спин-фаза (несколько итераций CAS) ...
    // не получилось → засыпаем:
    runtime_SemacquireMutex(&m.sema, ...)
}

// sync.Mutex.Unlock():
func (m *Mutex) unlockSlow() {
    runtime_Semrelease(&m.sema, ...)  // будим ожидающую горутину
}
```

^rsem-mutex-usage

```go
// sync.WaitGroup.Wait():
runtime_Semacquire(&wg.sema)  // засыпаем пока counter > 0

// sync.WaitGroup.Done() → Add(-1):
runtime_Semrelease(&wg.sema, ...)  // будим Wait() когда counter == 0
```

^rsem-wg-usage

### Stack trace

Когда видишь в профиле или panic:

```
goroutine 42 [semacquire]:
sync.runtime_SemacquireMutex(0xc0000b4004, 0x0, 0x1)
    /usr/local/go/src/runtime/sema.go:71
sync.(*Mutex).lockSlow(0xc0000b4000)
```

^rsem-stacktrace

Это значит: горутина **заблокирована** на мьютексе, ждёт пока кто-то сделает Unlock → Semrelease. ^rsem-stacktrace-meaning

### Внутренняя структура: semtable

Runtime хранит **глобальную хэш-таблицу** `semtable[251]` — по адресу семафора находит очередь ожидающих (treap — дерево + куча). ^rsem-semtable

```
semtable:
  hash(addr1) → treap: [sudog_A, sudog_B]  (ждут этот мьютекс)
  hash(addr2) → treap: [sudog_C]            (ждут этот WaitGroup)
```

^rsem-semtable-structure

### Fast path vs slow path

**Fast path**: `*s > 0` → атомарный декремент, **без** системных вызовов, без переключения контекста. Наносекунды. ^rsem-fast

**Slow path**: горутина засыпает (gopark) → планировщик переключается. Микросекунды. ^rsem-slow

Поэтому Mutex сначала **спинит** (CAS, fast path) и только потом идёт в Semacquire (slow path). ^rsem-spin-then-sleep

---

## gopark / goready

Парковка и пробуждение горутин. Самый низкий уровень управления состоянием G. ^gopark-mechanism

```
gopark(unlockf, lock, reason, ...):
  1. G.status: _Grunning → _Gwaiting
  2. Вызвать unlockf (отпустить лок атомарно с засыпанием)
  3. Отвязать G от M → schedule() (M берёт другую G из очереди)

goready(gp, traceskip):
  1. G.status: _Gwaiting → _Grunnable
  2. Положить G в локальную очередь P → может сразу побежать
```

^gopark-flow

### Где используется

|Кто вызывает|gopark (засыпает)|goready (будит)|
|:--|:--|:--|
|**Каналы**|`ch <- x` при полном буфере / `<-ch` при пустом|отправитель/получатель на другой стороне|
|**Select**|ждёт любой из каналов|первый готовый канал|
|**Таймеры**|`time.Sleep`, `time.After`|timer heap при срабатывании|
|**Netpoll**|`conn.Read()` — нет данных|epoll/kqueue сообщил о готовности|
|**Semacquire**|slow path мьютекса/WG|Semrelease при Unlock/Done|
|^gopark-where|||

### Stack trace

```
goroutine 15 [chan send]:           ← gopark с reason "chan send"
goroutine 22 [select]:             ← gopark с reason "select"
goroutine 8 [IO wait]:             ← gopark с reason "IO wait" (netpoll)
goroutine 31 [sleep]:              ← gopark с reason "sleep"
```

^gopark-reasons

Строка в скобках — это **reason** из gopark. По ней сразу видно, **на чём** горутина заблокирована. ^gopark-reason-meaning

### Ключевое отличие от Semacquire

`gopark` — **примитив** (припарковать одну G). `Semacquire` — **абстракция** поверх gopark (очередь ожидающих + счётчик). Каналы вызывают gopark **напрямую**, мьютексы — через Semacquire. ^gopark-vs-sem

---

## runtime_procPin / runtime_procUnpin

Привязать текущую горутину к текущему P. Пока привязана — планировщик **не перебросит** её на другой P. ^procpin-mechanism

```
procPin():
  1. Отключить preemption для текущей G
  2. Вернуть номер текущего P (pid)

procUnpin():
  1. Включить preemption обратно
```

^procpin-flow

### Где используется

**sync.Pool** — главный потребитель. Pool хранит **P-локальные** кэши. Чтобы безопасно обратиться к кэшу своего P, нужно гарантировать что тебя не перекинут на другой P посреди операции. ^procpin-pool

```go
// sync.Pool.Get() упрощённо:
func (p *Pool) Get() any {
    pid := runtime_procPin()   // привязались к P
    l := p.local[pid]          // безопасно: мы точно на этом P
    x := l.private             // P-локальный кэш, без мьютекса
    l.private = nil
    runtime_procUnpin()        // отвязались
    if x == nil {
        x = p.getSlow()       // fallback: shared pool, другие P
    }
    return x
}
```

^procpin-pool-usage

**Зачем**: P-локальный доступ **без мьютекса**. Если бы горутину перебросили на другой P между `local[pid]` и чтением `private` — прочитали бы чужой кэш. ^procpin-why

---

## runtime_canSpin / runtime_doSpin

Спин-фаза мьютекса перед засыпанием. ^spin-rt-def

```
runtime_canSpin(iter):
  - iter < 4 (не больше 4 итераций спина)
  - GOMAXPROCS > 1 (на одном ядре спинить бессмысленно)
  - Есть свободный P (иначе спин отбирает CPU у полезных горутин)

runtime_doSpin():
  - Выполнить 30 итераций PAUSE (процессорная инструкция)
  - PAUSE: подсказка CPU что это spin-wait → снижает энергопотребление
```

^spin-rt-flow

Если `canSpin` вернул false → горутина идёт в `Semacquire` (засыпает). ^spin-rt-fallback

---

## runtime_nanotime / runtime_walltime

`runtime_nanotime()` — **монотонные часы** (для измерения длительности, `time.Since`). Не прыгают при синхронизации NTP. ^time-nanotime

`runtime_walltime()` — **wall clock** (для `time.Now`). Может прыгать при NTP sync. ^time-walltime

---

## Сводка: кто кого вызывает

```
sync.Mutex.Lock()
  └→ atomic CAS (fast path — спин)
  └→ runtime_canSpin / runtime_doSpin (спин-фаза)
  └→ runtime_SemacquireMutex (slow path)
       └→ gopark (горутина засыпает)

sync.Mutex.Unlock()
  └→ runtime_Semrelease
       └→ goready (будим горутину)

ch <- x (полный буфер)
  └→ gopark напрямую (без Semacquire)

sync.Pool.Get()
  └→ runtime_procPin / runtime_procUnpin
```

^rt-call-tree

## Связь

- [[Mutex]] — Mutex.Lock/Unlock вызывает Semacquire/Semrelease
- [[Semaphore]] — runtime_Sem* = внутренняя реализация семафора
- [[Spinlock]] — Mutex спинит (canSpin/doSpin) перед Semacquire
- [[Contention]] — semacquire в стектрейсе = горутина заблокирована = contention
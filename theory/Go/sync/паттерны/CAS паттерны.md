- CAS (Compare-And-Swap) — атомарная инструкция: если значение == ожидаемое, заменить на новое. Иначе — ничего
- Основа всех lock-free структур: спинлоки, атомарные счётчики, lock-free очереди
- Паттерн: загрузить → вычислить → CAS → если не вышло, повторить (CAS loop)

---

## Инструкция

```
CAS(addr, old, new):
    атомарно:
        если *addr == old:
            *addr = new
            return true
        иначе:
            return false
```

На x86 — инструкция `CMPXCHG`. На ARM — пара `LDXR`/`STXR` (LL/SC). Гарантируется аппаратно. ^cas-hw-impl

## CAS loop — базовый паттерн

Почти всегда CAS используется в цикле:

```go
// Атомарный инкремент (без sync/atomic.AddInt64)
func AtomicAdd(addr *atomic.Int64, delta int64) int64 {
    for {
        old := addr.Load()
        new := old + delta
        if addr.CompareAndSwap(old, new) {
            return new
        }
        // кто-то изменил между Load и CAS — повторяем
    }
}
```

**Почему цикл**: между `Load()` и `CAS()` другой поток мог изменить значение. CAS обнаружит это (old ≠ текущее) и вернёт false. Повторяем. ^cas-loop-why

## ABA проблема

```
Поток 1: Load() → видит A
Поток 2: меняет A→B→A
Поток 1: CAS(A, new) → успех! Но значение уже "другое A"
```

^aba-problem

Решения:
- **Версионный счётчик**: CAS не на значение, а на пару (значение, версия). Каждое изменение увеличивает версию ^aba-solution-version
- **Hazard pointers** / **epoch-based reclamation**: для lock-free структур с указателями ^aba-solution-hazard
- В Go: `atomic.Pointer[T]` + счётчик поколений ^aba-solution-go

## Паттерны использования

**1. Lock-free счётчик:**
```go
var counter atomic.Int64
counter.Add(1)  // внутри — CAS loop
```

^cas-pattern-counter

**2. Lock-free флаг (one-shot):**
```go
var done atomic.Bool
if done.CompareAndSwap(false, true) {
    // только один поток выполнит это
    cleanup()
}
```

^cas-pattern-oneshot

**3. Lock-free обновление структуры (copy-on-write):**
```go
var config atomic.Pointer[Config]

func UpdateConfig(fn func(*Config) *Config) {
    for {
        old := config.Load()
        new := fn(old)      // создаём копию с изменениями
        if config.CompareAndSwap(old, new) {
            return
        }
    }
}
```

^cas-pattern-cow

**4. Spin lock:**
```go
for !locked.CompareAndSwap(false, true) {
    runtime.Gosched()
}
```

^cas-pattern-spinlock

**5. Lock-free стек (push):**
```go
func Push(head *atomic.Pointer[Node], val int) {
    node := &Node{val: val}
    for {
        old := head.Load()
        node.next = old
        if head.CompareAndSwap(old, node) {
            return
        }
    }
}
```

^cas-pattern-stack

## CAS vs Мьютекс

```
CAS (lock-free)           Мьютекс
Нет блокировки            Блокировка (park)
Retry при конфликте       Очередь ожидания
Хорош при низком          Хорош при высоком
  contention                contention
Сложнее корректность      Проще рассуждать
```

^cas-vs-mutex

При **высоком contention** CAS loop деградирует: много бесполезных retry, cache line bouncing. Мьютекс лучше — потоки спят вместо кручения. ^cas-high-contention

## Связь
- [[Spin lock]] — CAS loop для захвата лока
- [[Мьютекс Петерсона]] — мьютекс без CAS (исторический)
- [[True sharing и False sharing]] — cache line bouncing при CAS
- [[Внутреннее устройство очередей]] — CAS при краже из LRQ
- [[Стек Трайбера]], [[Очередь Майкла-Скотта]], [[ABA-проблема]]

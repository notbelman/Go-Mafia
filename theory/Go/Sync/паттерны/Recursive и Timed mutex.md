- Recursive mutex — мьютекс который один поток может захватить **повторно** без дедлока (счётчик вложенности)
- Timed mutex — мьютекс с **таймаутом**: не получил за N времени → отказ, а не вечное ожидание
- В Go **нет** ни того, ни другого в stdlib — и это сознательное решение

---

## Recursive mutex

Обычный мьютекс: если владелец вызовет Lock() повторно — **дедлок**. Recursive mutex разрешает повторный захват тому же потоку. ^recursive-def

```go
type RecursiveMutex struct {
    mu      sync.Mutex
    owner   int64          // ID владельца (goroutine ID)
    count   int32          // глубина вложенности
}

func (r *RecursiveMutex) Lock() {
    gid := getGoroutineID()
    if atomic.LoadInt64(&r.owner) == gid {
        r.count++  // уже наш — просто увеличиваем счётчик
        return
    }
    r.mu.Lock()
    atomic.StoreInt64(&r.owner, gid)
    r.count = 1
}

func (r *RecursiveMutex) Unlock() {
    r.count--
    if r.count == 0 {
        atomic.StoreInt64(&r.owner, 0)
        r.mu.Unlock()
    }
}
```

^recursive-impl

**Проблема в Go**: у горутин нет публичного ID. `runtime.Stack()` хак работает, но медленно. Это одна из причин почему Go не предоставляет recursive mutex. ^recursive-no-goid

**Почему Go против:**
- Recursive mutex скрывает плохой дизайн — если нужен рекурсивный лок, значит API запутанный ^recursive-bad-design
- Неясно сколько раз нужно Unlock() — ошибки гарантированы ^recursive-unlock-errors
- Лучше: выделить внутреннюю функцию без лока, внешнюю с локом ^recursive-go-alternative

## Timed mutex

Попытка захватить лок с дедлайном:

```go
type TimedMutex struct {
    ch chan struct{}
}

func NewTimedMutex() *TimedMutex {
    ch := make(chan struct{}, 1)
    ch <- struct{}{}
    return &TimedMutex{ch: ch}
}

func (t *TimedMutex) Lock(timeout time.Duration) bool {
    select {
    case <-t.ch:
        return true   // захватили
    case <-time.After(timeout):
        return false  // таймаут — не ждём вечно
    }
}

func (t *TimedMutex) Unlock() {
    t.ch <- struct{}{}
}
```

^timed-impl

**Зачем:**
- Защита от дедлоков: если лок не получен за X мс — откатываемся ^timed-why-deadlock
- Graceful degradation: лучше отказ чем зависание ^timed-why-graceful
- Полезно в сетевых сервисах с SLA ^timed-why-sla

**Go-идиоматичный подход:** вместо timed mutex используют `context.WithTimeout` + select:

```go
func doWork(ctx context.Context) error {
    select {
    case token <- lockChan:
        defer func() { lockChan <- token }()
        // работаем
    case <-ctx.Done():
        return ctx.Err()  // таймаут
    }
}
```

^timed-go-idiomatic

## Почему в Go нет обоих

- Recursive mutex: `sync.Mutex` намеренно не рекурсивный. Философия Go: простой API, явные контракты ^go-no-recursive
- Timed mutex: решается через каналы + контексты — более гибко ^go-no-timed

## Связь
- [[Примитивный мьютекс]] — базовый мьютекс без рекурсии и таймаутов
- [[Семафор]] — timed mutex = семафор на 1 с таймаутом
- [[Spin lock]] — тоже нет рекурсии и таймаутов

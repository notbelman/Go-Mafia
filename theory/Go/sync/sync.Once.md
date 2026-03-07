Гарантирует выполнение функции ровно один раз (Do(f)). ^once-definition

**Fast path:** `done == true` → return ^once-fast-path

**Slow path:**
1. Захват mutex
2. Double-check `done`
3. Выполнение `f()`
4. `done = true`
^once-slow-path

**Почему mutex, а не CAS?**
CAS не блокирует — вторая горутина уйдёт сразу, не дождавшись `f()`. Mutex заставляет всех ждать завершения. ^once-why-mutex-not-cas

Рекурсивный `Do()` внутри `f()` — deadlock (mutex не reentrant). ^once-recursive-deadlock

```go
var once sync.Once
var config *Config

func GetConfig() *Config {
    once.Do(func() {
        config = loadConfig()
    })
    return config
}
```
^once-usage-example

---

## Внутреннее устройство
```go
type Once struct {
    _    noCopy      // защита от копирования
    done atomic.Bool // флаг выполнения (первое поле -- hot path, меньше инструкций)
    m    Mutex       // для блокировки конкурентных вызовов
}
```
^once-struct

### Почему `done` — первое поле в структуре

`done` стоит первым, потому что это hot path: на x86 доступ к первому полю структуры требует меньше инструкций (нет offset). Это ускоряет fast path. ^once-done-first-field

### Алгоритм Do()

```go
func (o *Once) Do(f func()) {
    if !o.done.Load() {      // fast path: уже выполнено?
        o.doSlow(f)
    }
}

func (o *Once) doSlow(f func()) {
    o.m.Lock()
    defer o.m.Unlock()
    if !o.done.Load() {      // double-check под локом
        defer o.done.Store(true)
        f()
    }
}
```
^once-algorithm

## Связь
- [[sync.Cond]] — другой sync-примитив
- [[Atomic vs Mutex]] — Once: fast path atomic, slow path mutex
- [[CAS и CAS loop]] — done.Load() = atomic fast path

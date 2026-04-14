- CAS (CompareAndSwap): если *addr == old → *addr = new, return true; иначе false. Одна CPU инструкция ^cas-definition
- CAS loop: Load → compute → CAS → если не прошёл, повторить. Основа lock-free структур данных ^cas-loop-concept
- ошибка: Load + Store раздельно = race. CAS решает: read-modify-write атомарно ^cas-vs-load-store

---

## Псевдокод CAS

```go
// Это НЕ реальный код — просто логика того, что делает CAS атомарно
func CompareAndSwap(addr *int32, old, new int32) bool {
    if *addr == old {
        *addr = new
        return true
    }
    return false
}
```

Вся эта логика выполняется **одной CPU инструкцией** (LOCK CMPXCHG на x86). Между проверкой и записью никто не вклинится. ^cas-cpu-instruction

## Ошибка: Load + Store раздельно

```go
var initialized atomic.Bool

// ПЛОХО: между Load и Store может вклиниться другая горутина
func init() {
    if !initialized.Load() {    // обе горутины прочитали false
        initialized.Store(true) // обе записали true
        m = make(map[string]int) // map создалась дважды!
    }
}

// ХОРОШО: CAS = read + compare + write атомарно
func init() {
    if initialized.CompareAndSwap(false, true) {
        m = make(map[string]int) // только одна горутина зайдёт
    }
}
```

^cas-load-store-race

## CAS loop

```go
func IncrementAndGet(addr *int64) int64 {
    for {
        current := atomic.LoadInt64(addr)       // 1. атомарно читаем
        next := current + 1                      // 2. вычисляем (локально)
        if atomic.CompareAndSwapInt64(addr, current, next) {
            return next                          // 3. CAS прошёл — готово
        }
        // CAS не прошёл — кто-то изменил между Load и CAS
        // повторяем с новым current
    }
}
```

^cas-loop-code

На CAS loop основаны lock-free структуры данных: очередь Майкла-Скотта, стек Трайбера. ^cas-lock-free-structures

Гарантия прогресса есть — рано или поздно CAS пройдёт. ^cas-progress-guarantee

## Когда CAS НЕ нужен

```go
// ИЗБЫТОЧНО: CAS loop для инкремента
// atomic.Add уже делает это за нас и возвращает новое значение

value := atomic.AddInt64(&counter, 1)
if value % 100 == 0 {
    // делаем что-то каждые 100 инкрементов
    // value — локальное, проверять безопасно
}
```

Если atomic.Add возвращает новое значение — проверяйте его локально, CAS не нужен. ^cas-not-needed

## Связь
- [[sync - atomic]] — API атомарных операций
- [[Atomic vs Mutex]] — атомики vs мьютексы
- [[Почему НЕ atomic везде]] — ограничения lock-free подхода
- [[sync.Once]] — внутри использует mutex, а не CAS (чтобы вторая горутина дождалась завершения f())

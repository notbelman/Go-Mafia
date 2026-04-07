#flashcards/sync-patterns/spurious_wakeup

Что такое spurious wakeup? Разрешён ли он спецификацией?
?
![[Spurious_wakeup#^spurious-def]]

Назови три причины почему spurious wakeup существует.
?
![[Spurious_wakeup#^spurious-reason-os]]
![[Spurious_wakeup#^spurious-reason-perf]]
![[Spurious_wakeup#^spurious-reason-simplicity]]

Почему для ожидания на condition variable нужен for, а не if? Покажи оба варианта.
?
![[Spurious_wakeup#^spurious-for-pattern]]

Что такое stolen wakeup? Опиши сценарий по шагам.
?
![[Spurious_wakeup#^stolen-wakeup]]

Подвержены ли spurious wakeup каналы в Go? А sync.Cond? А select?
?
![[Spurious_wakeup#^go-channels-safe]]
![[Spurious_wakeup#^go-synccond-spurious]]
![[Spurious_wakeup#^go-select-safe]]

Что гарантирует Go runtime при пробуждении через `<-ch`?
?
![[Spurious_wakeup#^go-channels-safe]]

Что говорит документация Go про sync.Cond.Wait()?
?
![[Spurious_wakeup#^go-synccond-spurious]]

Что выведет этот код?
```go
var mu sync.Mutex
var cond = sync.NewCond(&mu)
var ready bool

go func() {
    mu.Lock()
    if !ready {          // IF вместо FOR
        cond.Wait()
    }
    fmt.Println("worker done")
    mu.Unlock()
}()

time.Sleep(10 * time.Millisecond)
mu.Lock()
ready = true
mu.Unlock()
cond.Signal()

time.Sleep(50 * time.Millisecond)
```
?
Обычно выводит `worker done`, но код некорректен: использует `if` вместо `for`. При spurious wakeup горутина проснётся до того как `ready == true` и продолжит выполнение с невыполненным условием. Правильный паттерн: `for !ready { cond.Wait() }`.
![[Spurious_wakeup#^spurious-for-pattern]]

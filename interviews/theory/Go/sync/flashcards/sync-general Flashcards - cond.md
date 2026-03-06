#flashcards/sync-general/cond

Для чего используется sync.Cond? Опиши в одном предложении.
?
![[sync.Cond#^cond-definition]]

Что делает Wait() под капотом — опиши три шага?
?
![[sync.Cond#^cond-api-table]]

Почему Wait() нужно вызывать в цикле `for`, а не в `if`?
?
![[sync.Cond#^cond-wait-for-loop]]

Чем Signal() отличается от Broadcast()? Что происходит если вызвать Broadcast() несколько раз?
?
![[sync.Cond#^cond-signal-vs-broadcast]]

Что произойдёт если вызвать cond.Wait() без предварительного захвата лока?
?
Это UB / некорректное использование. Правило: Wait() вызывать только с захваченным `cond.L`. Wait() атомарно отпускает лок при засыпании и захватывает при пробуждении. Нарушение — data race.
![[sync.Cond#^cond-api-table]]

Опиши внутреннюю структуру sync.Cond. Что такое notifyList?
?
![[sync.Cond#^cond-internals]]

Что выведет этот код?
```go
var mu sync.Mutex
cond := sync.NewCond(&mu)
ready := false

go func() {
    mu.Lock()
    ready = true
    mu.Unlock()
    cond.Signal()
}()

mu.Lock()
if !ready {
    cond.Wait()
}
mu.Unlock()
fmt.Println("done")
```
?
Скорее всего `done`, но код содержит баг: `if` вместо `for`. Если горутина выполнится до того как main войдёт в `if`, `ready == true` и Wait не вызовется — всё ок. Но при spurious wakeup или смене порядка — баг. Нужно `for !ready { cond.Wait() }`.
![[sync.Cond#^cond-wait-for-loop]]

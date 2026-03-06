#flashcards/channels-patterns/done_channel

Что такое паттерн Done channel? Какую проблему он решает по сравнению с context?
?
![[Done channel#^done-problem]]

Сколько каналов в done channel паттерне и какую роль играет каждый?
?
![[Done channel#^done-func-impl]]

Что выведет этот код? Завершится ли программа корректно?
```go
closeCh := make(chan struct{})
closedCh := doWork(closeCh)
time.Sleep(500 * time.Millisecond)
close(closeCh)
<-closedCh
fmt.Println("done")
```
?
Напечатает `done` — паттерн гарантирует завершение. `close(closeCh)` посылает сигнал горутине, `<-closedCh` блокируется до тех пор пока горутина не выполнит `defer close(closedCh)` и не вернётся.
![[Done channel#^done-func-impl]]

Как реализовать Shutdown() для воркера через done channel?
?
![[Done channel#^done-worker-impl]]

Сравни контекст и done channel: что каждый умеет, сколько каналов, когда применять.
?
![[Done channel#^done-vs-context]]

Почему нельзя смешивать context.Context и done channel?
?
![[Done channel#^done-no-mix]]

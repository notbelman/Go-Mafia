#flashcards/channels-other/close

Что произойдёт если вызвать `close(nil)`?
?
![[Закрытие канала#^close-nil-panic]]

Что произойдёт если вызвать `close(ch)` на уже закрытом канале?
?
![[Закрытие канала#^close-already-closed]]

Перечисли шаги, которые выполняет runtime при вызове `close(ch)` (8 шагов).
?
![[Закрытие канала#^close-set-flag]]
![[Закрытие канала#^close-recvq]]
![[Закрытие канала#^close-sendq]]
![[Закрытие канала#^close-goready-all]]

Что получат горутины из `recvq` при закрытии канала?
?
![[Закрытие канала#^close-recvq]]

Что случится с горутинами из `sendq` при закрытии канала? Паника происходит сразу или нет?
?
![[Закрытие канала#^close-sendq-deferred-panic]]

Закрытый канал — что можно, что нельзя?
?
![[Закрытие канала#^close-read-write-rule]]

Почему `close(ch)` — единственный способ broadcast через каналы?
?
![[Закрытие канала#^close-broadcast]]

В чём ключевое отличие `close(ch)` от `sync.Cond.Broadcast`?
?
![[Закрытие канала#^close-vs-synccond]]

Буферизированный канал: `make(chan int, 2)`, кладём 10, 20, потом `close`. Что вернут три последовательных чтения?
?
![[Закрытие канала#^close-buffered-read]]

Что выведет этот код?
```go
ch := make(chan int)
go func() {
    ch <- 42
}()
time.Sleep(500 * time.Millisecond)
close(ch)
time.Sleep(100 * time.Millisecond)
```
?
`panic: send on closed channel` — горутина в sendq просыпается при close и обнаруживает закрытый канал. Паника происходит не в момент close, а при попытке продолжить send.
![[Закрытие канала#^close-blocked-writers]]

Что выведет этот код?
```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
close(ch)
for v := range ch {
    fmt.Println(v)
}
fmt.Println("done")
```
?
`1`, `2`, `done` — `range` по закрытому каналу дочитывает все данные из буфера, потом завершается. Данные в буфере не уничтожаются при close.
![[Закрытие канала#^close-buffered-read]]

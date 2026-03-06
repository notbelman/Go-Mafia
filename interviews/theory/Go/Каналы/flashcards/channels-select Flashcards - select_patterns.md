#flashcards/channels-select/select_patterns

Что делает `select {}` (пустой select)? Это валидный код? Как ведёт себя рантайм?
?
![[select - Специфические состояния и паттерны#^select-empty]]

Почему `select {}` не нагружает CPU, хотя блокирует горутину навсегда?
?
![[select - Специфические состояния и паттерны#^select-empty]]

Как реализовать таймаут на операцию с каналом в Go?
?
![[select - Специфические состояния и паттерны#^select-timeout-pattern]]

Опиши паттерн Graceful Shutdown с использованием `select`.
?
![[select - Специфические состояния и паттерны#^select-graceful-shutdown]]

Чем паттерн `for-select` с `ctx.Done()` отличается от `for-select` с каналом `done`?
?
![[select - Специфические состояния и паттерны#^select-graceful-shutdown]]

Что выведет этот код? Что произойдёт через секунду?
```go
ch := make(chan int)
go func() {
    time.Sleep(2 * time.Second)
    ch <- 42
}()
select {
case v := <-ch:
    fmt.Println("got", v)
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
}
```
?
`timeout` — `time.After` срабатывает через 1 секунду, горутина ещё не успела отправить (2 секунды). `select` выбирает готовый `case` с таймером.
![[select - Специфические состояния и паттерны#^select-timeout-pattern]]

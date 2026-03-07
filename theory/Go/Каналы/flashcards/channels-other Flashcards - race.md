#flashcards/channels-other/race

Назови 3 типичных race condition с каналами.
?
![[Race condition в каналах#^race-cases]]

Почему `if len(ch) > 0 { v := <-ch }` — это race condition?
?
![[Race condition в каналах#^race-len-toctou]]

Что произойдёт если два concurrent `close(ch)` выполнятся на одном канале?
?
![[Race condition в каналах#^race-double-close]]

Как безопасно закрыть канал когда несколько горутин могут вызвать `close`?
?
![[Race condition в каналах#^race-double-close]]

Что произойдёт при concurrent `ch <- 1` и `close(ch)`?
?
![[Race condition в каналах#^race-send-close]]

Сформулируй главное правило владения каналом, которое предотвращает все race conditions.
?
![[Race condition в каналах#^race-ownership-rule]]

Что выведет этот код?
```go
ch := make(chan int, 1)
go func() { close(ch) }()
go func() { close(ch) }()
time.Sleep(100 * time.Millisecond)
```
?
`panic: close of closed channel` — два concurrent close на одном канале. close не идемпотентен. Решение: `sync.Once` или один ответственный.
![[Race condition в каналах#^race-double-close]]

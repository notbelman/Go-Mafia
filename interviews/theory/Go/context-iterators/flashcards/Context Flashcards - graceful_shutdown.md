#flashcards/context/graceful_shutdown

Что такое signal.NotifyContext? Чем он отличается от обычного WithCancel?
?
![[Graceful Shutdown#^gs-notify-context]]

Разработчик написал: «У меня есть graceful shutdown — я слушаю signal.NotifyContext и жду <-ctx.Done()». Что не так?
?
![[Graceful Shutdown#^gs-not-graceful]]

Дай точное определение: что такое graceful shutdown? Из каких шагов состоит?
?
![[Graceful Shutdown#^gs-definition]]

Чем srv.Shutdown(ctx) отличается от srv.Close()?
?
![[Graceful Shutdown#^gs-shutdown-vs-close]]

Почему при graceful shutdown используют отдельный context.WithTimeout для srv.Shutdown, а не тот же ctx от signal.NotifyContext?
?
![[Graceful Shutdown#^gs-shutdown-timeout]]

Опиши полный паттерн graceful shutdown HTTP-сервера в Go по шагам.
?
![[Graceful Shutdown#^gs-pattern]]

Что выведет этот код при получении SIGINT?
```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
defer stop()
<-ctx.Done()
fmt.Println("got signal")
// программа завершается
```
?
`got signal` — ctx отменится при SIGINT, <-ctx.Done() разблокируется. Но это НЕ graceful shutdown: активные HTTP-коннекшены не дождутся завершения, ресурсы не освобождены явно.
![[Graceful Shutdown#^gs-not-graceful]]

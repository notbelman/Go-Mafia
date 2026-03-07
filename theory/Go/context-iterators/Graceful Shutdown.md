- signal.NotifyContext — создаёт контекст, который отменяется по сигналу ОС (SIGINT, SIGTERM)
- «создать signal context ≠ graceful shutdown» — после отмены нужно ЯВНО тушить воркеров, флашить логи, закрывать коннекшены
- server.Shutdown (graceful, ждёт коннекшены) vs server.Close (immediate, рубит всё)

---

## Паттерн

```go
func main() {
    // 1. Контекст, который отменится по Ctrl+C / kill
    ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
    defer stop()

    srv := &http.Server{Addr: ":8080", Handler: mux}

    // 2. Сервер в отдельной горутине
    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    // 3. Ждём сигнал отмены
    <-ctx.Done()
    log.Println("shutting down...")

    // 4. ВОТ ЭТО и есть graceful shutdown:
    shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    srv.Shutdown(shutdownCtx)  // ждёт завершения активных коннекшенов
    flushLogs()                // дописываем логи
    closeDB()                  // закрываем БД
    log.Println("done")
}
```

^gs-pattern

## signal.NotifyContext

`signal.NotifyContext` — создаёт контекст, который отменяется при получении сигнала ОС (SIGINT, SIGTERM). По сути это WithCancel + слушатель сигналов. ^gs-notify-context

## Критическое заблуждение: signal context ≠ graceful shutdown

Создать signal context и дождаться `<-ctx.Done()` — это только signal handling, не graceful shutdown. ^gs-not-graceful

Graceful shutdown = получил сигнал → дождался завершения текущих операций → освободил ресурсы → вышел. Без явного тушения воркеров это просто ожидание сигнала. ^gs-definition

## Типичная ошибка: «у меня graceful shutdown»

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
defer stop()
<-ctx.Done()
// ... и всё? Где shutdown? Это просто ожидание сигнала, не graceful shutdown
```

^gs-mistake-pattern

## Shutdown vs Close

| Метод | Что делает |
|:------|:-----------|
| `srv.Shutdown(ctx)` | перестаёт принимать новые, ждёт активные коннекшены |
| `srv.Close()` | рубит всё немедленно |

^gs-shutdown-vs-close

## Таймаут на shutdown

После получения сигнала используется отдельный контекст с таймаутом (`context.WithTimeout`) для самого shutdown — чтобы не ждать вечно, если коннекшен завис. ^gs-shutdown-timeout

## Связь
- [[context WithCancel]] — signal.NotifyContext = WithCancel + слушатель сигналов
- [[context WithTimeout]] — таймаут на сам shutdown (чтобы не ждать вечно)
- [[Context]] — дерево контекстов: один корневой → все дочерние отменяются

Плавное завершение работы: дождаться текущие запросы, корректно закрыть ресурсы, не терять данные.

**Ключевые моменты:**
- SIGINT (Ctrl+C), SIGTERM (kill, Kubernetes) -- можно перехватить
- SIGKILL (kill -9) -- нельзя перехватить
- `signal.Notify` или `signal.NotifyContext` для перехвата
- `http.Server.Shutdown(ctx)` -- ждёт текущие запросы
- `sync.WaitGroup` -- для своих горутин
- таймаут на shutdown -- не ждать вечно
- порядок: перестать принимать → дождаться текущее → закрыть ресурсы

**пример:**
```go
func main() {
    // 1. context отменится при SIGINT/SIGTERM
    ctx, stop := signal.NotifyContext(context.Background(),
        syscall.SIGINT, syscall.SIGTERM)
    defer stop()

    srv := &http.Server{Addr: ":8080", Handler: http.DefaultServeMux}

    // 2. сервер в горутине
    go srv.ListenAndServe()

    // 3. ждём сигнал (ctx.Done() закроется при Ctrl+C)
    <-ctx.Done()

    // 4. даём 5 секунд на завершение запросов
    shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    srv.Shutdown(shutdownCtx)
}
```

**С WaitGroup для своих горутин:**
```go
var wg sync.WaitGroup

// запуск воркера
wg.Add(1)
go func() {
    defer wg.Done()
    for {
        select {
        case <-ctx.Done():
            return  // завершаемся по сигналу
        case job := <-jobs:
            process(job)
        }
    }
}()

// при shutdown
<-quit
cancel()           // сигнал горутинам
srv.Shutdown(ctx)  // ждём HTTP
wg.Wait()          // ждём свои горутины
db.Close()         // закрываем ресурсы
```

**http.Server.Shutdown:**
```go
// что делает Shutdown:
// 1. закрывает listeners (перестаёт принимать новые)
// 2. закрывает idle connections
// 3. ждёт active connections до завершения или таймаута
// 4. НЕ закрывает hijacked connections (WebSocket)

// для WebSocket нужен RegisterOnShutdown:
srv.RegisterOnShutdown(func() {
    // уведомить WebSocket клиентов
})
```

**Kubernetes:**
```
SIGTERM → ждёт terminationGracePeriodSeconds (30s) → SIGKILL
```

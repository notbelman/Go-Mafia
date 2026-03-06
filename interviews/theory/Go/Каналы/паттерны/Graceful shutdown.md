- Graceful shutdown — завершение приложения с дожиданием текущей работы (коннекшены, flush на диск, последние запросы)
- os.Signal → канал → select с приоритизацией → WaitGroup для ожидания воркеров
- Две стратегии: воркеры сами слушают done-канал, или main получает сигнал и тушит всё явно

---

## Зачем

Без graceful shutdown: ^gs-why

- Клиенты получают 5xx на полпути
- Данные не дописываются на диск
- Файловые дескрипторы не закрываются
- Логи теряются

## Реализация через каналы

```go
func main() {
    // 1. Канал для сигналов OS
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, syscall.SIGINT, syscall.SIGTERM)

    // 2. Воркер с WaitGroup
    var wg sync.WaitGroup
    wg.Add(1)

    go func() {
        defer wg.Done()
        ticker := time.NewTicker(1 * time.Second)
        defer ticker.Stop()

        for {
            select {
            case <-sigCh:           // приоритет: сигнал завершения
                fmt.Println("worker stopping")
                return
            case <-ticker.C:
                fmt.Println("working...")
            }
        }
    }()

    // 3. Ждём завершения всех воркеров
    wg.Wait()
    fmt.Println("application stopped")
}
```

^gs-impl

## Две стратегии

**Стратегия 1 — воркеры слушают сигнал:** Каждый воркер в select слушает done-канал. При сигнале — завершает текущую работу и выходит. Main ждёт через WaitGroup. ^gs-strategy1

**Стратегия 2 — main тушит явно:** Main получает сигнал и начинает последовательно: закрывать коннекшены, flush буферов, закрывать файлы, shutdown HTTP-сервера. ^gs-strategy2

```go
<-sigCh
server.Shutdown(ctx)    // дожидаемся текущих запросов
db.Close()
logFile.Sync()
logFile.Close()
```

## signal.Notify

```go
sigCh := make(chan os.Signal, 1)  // буфер 1 обязателен!
signal.Notify(sigCh, syscall.SIGINT, syscall.SIGTERM)
```

- `SIGINT` — Ctrl+C
- `SIGTERM` — kill (дефолтный сигнал)
- `SIGKILL` — нельзя перехватить (kill -9)
- Буфер 1: чтобы сигнал не потерялся если никто ещё не читает ^gs-signals

## С контекстами

Контексты упрощают: `signal.NotifyContext` → контекст отменяется по сигналу. Но каналы дают больше контроля (например, двухфазный shutdown). ^gs-context

## Связь
- [[Done channel]] — паттерн двух каналов для отмены + подтверждения
- [[Ticker]] — часто используется в воркерах с graceful shutdown

- errgroup.WithContext: запускает группу горутин, ошибка в одной → отмена всех остальных ^eg-purpose
- Wait() блокируется до завершения всех горутин, возвращает первую ошибку ^eg-wait-behavior
- кейс: распределённые запросы к шардам — один упал, остальных ждать бессмысленно ^eg-usecase

---

## Паттерн

```go
g, ctx := errgroup.WithContext(parentCtx)

for i := 0; i < 10; i++ {
    shardID := i
    g.Go(func() error {
        // ctx отменится если любая горутина вернёт ошибку
        select {
        case <-ctx.Done():
            return ctx.Err()  // кто-то другой упал
        case <-time.After(queryTime):
            if shardID == 5 {
                return errors.New("shard 5 failed")
            }
            return nil
        }
    })
}

if err := g.Wait(); err != nil {
    // первая ошибка: "shard 5 failed"
    // остальные горутины получили ctx.Done() и завершились
    fmt.Println(err)
}
```

## Как это работает

errgroup.WithContext создаёт дочерний контекст с cancel. Когда любая горутина из g.Go() возвращает non-nil error, вызывается cancel() → ctx.Done() срабатывает у всех остальных горутин. ^eg-mechanism

Wait() ждёт ВСЕ горутины (не только до первой ошибки). ^eg-wait-all

Возвращает первую ошибку. Несколько горутин могут вернуть ошибки одновременно (если ctx.Done() и результат пришли в один момент), но Wait() вернёт только первую. ^eg-first-error

## Зачем, если можно WaitGroup + канал

Можно. Но errgroup инкапсулирует: создание контекста, cancel при ошибке, сбор первой ошибки, ожидание всех горутин. Меньше бойлерплейта, меньше шансов забыть cancel или утечь горутину. ^eg-vs-waitgroup

## Связь
- [[context WithCancel]] — errgroup использует WithCancel внутри
- [[Оборачивание функций без контекста]] — паттерн для одной горутины
- [[Graceful shutdown]] — errgroup можно использовать для координации shutdown

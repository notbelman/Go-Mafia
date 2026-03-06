#flashcards/context/errgroup

Что делает errgroup.WithContext? Опиши механизм по шагам.
?
![[errgroup с контекстами#^eg-purpose]]
![[errgroup с контекстами#^eg-mechanism]]

Как ведёт себя errgroup.Wait()? Когда он возвращается и что возвращает?
?
![[errgroup с контекстами#^eg-wait-behavior]]
![[errgroup с контекстами#^eg-wait-all]]
![[errgroup с контекстами#^eg-first-error]]

Несколько горутин в errgroup упали почти одновременно. Что вернёт Wait()?
?
![[errgroup с контекстами#^eg-first-error]]

Назови типичный кейс где errgroup — правильное решение.
?
![[errgroup с контекстами#^eg-usecase]]

Зачем использовать errgroup вместо WaitGroup + канал? Что он инкапсулирует?
?
![[errgroup с контекстами#^eg-vs-waitgroup]]

Что выведет этот код (упрощённо)? Сколько горутин завершится и как?
```go
g, ctx := errgroup.WithContext(context.Background())

for i := 0; i < 3; i++ {
    id := i
    g.Go(func() error {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-time.After(100 * time.Millisecond):
            if id == 1 {
                return errors.New("shard 1 failed")
            }
            return nil
        }
    })
}

err := g.Wait()
fmt.Println(err)
```
?
`shard 1 failed` — Wait() ждёт ВСЕ 3 горутины. Горутина с id==1 вернула ошибку → cancel() → остальные горутины получают ctx.Done() и возвращают context.Canceled. Wait() возвращает первую ошибку.
![[errgroup с контекстами#^eg-mechanism]]
![[errgroup с контекстами#^eg-wait-all]]
![[errgroup с контекстами#^eg-first-error]]

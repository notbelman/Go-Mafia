#flashcards/context/wrap_no_ctx

Опиши паттерн оборачивания функции без поддержки контекста. Из каких частей он состоит?
?
![[Оборачивание функций без контекста#^wrap-pattern-overview]]

Почему канал результата в паттерне обёртки ОБЯЗАН быть буферизированным (cap=1)?
?
![[Оборачивание функций без контекста#^wrap-buffered-required]]
![[Оборачивание функций без контекста#^wrap-why-buffered]]

Опиши точный сценарий утечки горутины, если канал небуферизированный.
?
![[Оборачивание функций без контекста#^wrap-leak-scenario]]

Когда реально нужен паттерн оборачивания? Когда он не нужен?
?
![[Оборачивание функций без контекста#^wrap-when-needed]]

Что выведет этот код? Утечёт ли горутина?
```go
func call(ctx context.Context) (string, error) {
    ch := make(chan string) // без буфера!
    go func() {
        time.Sleep(5 * time.Second)
        ch <- "result"
    }()
    select {
    case r := <-ch:
        return r, nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
defer cancel()
res, err := call(ctx)
fmt.Println(res, err)
```
?
` context deadline exceeded` — ctx истёк через 100ms, select вернул ошибку. Горутина внутри продолжает спать 5 секунд, затем пытается записать в `ch` — но читателя нет → горутина навечно заблокирована. **Утечка.**
![[Оборачивание функций без контекста#^wrap-leak-scenario]]
![[Оборачивание функций без контекста#^wrap-buffered-required]]

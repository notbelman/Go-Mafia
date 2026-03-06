#flashcards/context/background_todo

Чем Background отличается от TODO семантически — оба пустые заглушки, в чём разница?
?
![[context Background и TODO#^bg-vs-todo-semantics]]

Где допустимо использовать context.Background()? Назови все случаи.
?
![[context Background и TODO#^bg-todo-usage]]

Где использовать context.TODO()? Что это сигнализирует команде?
?
![[context Background и TODO#^todo-definition]]

Почему передача nil вместо контекста вызывает панику, а не просто ошибку?
?
![[context Background и TODO#^nil-not-convention]]

Что конкретно произойдёт при вызове ctx.Err() или ctx.Done() если ctx == nil?
?
![[context Background и TODO#^nil-panic]]

Почему TODO на практике редко исправляют — и что из этого следует?
?
![[context Background и TODO#^todo-antipattern]]

Что выведет этот код?
```go
func doWork(ctx context.Context) {
    fmt.Println(ctx.Err())
}

func main() {
    doWork(context.Background())
    doWork(context.TODO())
}
```
?
`<nil>` и `<nil>` — оба контекста никогда не отменяются, Err() всегда nil.
![[context Background и TODO#^bg-definition]]

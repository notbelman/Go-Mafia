#flashcards/context/context

Какие 4 метода определяет интерфейс context.Context? Что возвращает каждый?
?
![[Context#^ctx-interface-def]]

Любой тип, реализующий интерфейс Context, является контекстом — что это означает на практике?
?
![[Context#^ctx-interface-def]]

Что произойдёт при отмене родительского контекста в дереве — затронет ли это соседей?
?
![[Context#^ctx-tree-cancel-down]]
<!--SR:!2026-02-27,3,250-->

Ты отменил db query (дочерний контекст). Что произойдёт с request (родителем) и http call (соседом)?
?
![[Context#^ctx-tree-cancel-isolated]]

Что означает "природа матрёшки" применительно к context?
?
![[Context#^ctx-propagation]]

Для чего предназначен context? Назови три ключевых возможности.
?
![[Context#^ctx-purpose]]

Что выведет этот код?
```go
ctx, cancel := context.WithCancel(context.Background())
go func() {
    <-ctx.Done()
    fmt.Println("child done:", ctx.Err())
}()
cancel()
time.Sleep(10 * time.Millisecond)
```
?
`child done: context canceled` — cancel() закрывает Done() канал, горутина разблокируется, Err() возвращает context.Canceled.
![[Context#^ctx-propagation]]

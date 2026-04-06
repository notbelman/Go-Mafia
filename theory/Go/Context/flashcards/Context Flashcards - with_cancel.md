#flashcards/context/with_cancel

Что происходит внутри при вызове cancel() — на уровне каналов и дочерних контекстов?
?
![[context WithCancel#^cancel-closes-channel]]

Безопасно ли вызывать cancel() несколько раз? Почему?
?
![[context WithCancel#^cancel-idempotent]]

Антипаттерн: зачем НЕ передавать cancel в другую функцию?
?
![[context WithCancel#^cancel-ownership]]

Паттерн fan-out с WithCancel: 10 горутин, нужна только первый результат. Опиши механизм.
?
![[context WithCancel#^cancel-fan-out-pattern]]

Почему нельзя слушать ctx.Done() последовательно после чтения из канала данных?
?
![[context WithCancel#^cancel-select-pattern]]

Что выведет этот код?
```go
ctx, cancel := context.WithCancel(context.Background())
cancel()
cancel()
cancel()
fmt.Println(ctx.Err())
```
?
`context canceled` — cancel() идемпотентен, канал закрывается ровно один раз, повторные вызовы безопасны.
![[context WithCancel#^cancel-idempotent]]

Что выведет этот код?
```go
parent, cancelParent := context.WithCancel(context.Background())
child, cancelChild := context.WithCancel(parent)
defer cancelChild()

cancelParent()
fmt.Println(child.Err())
```
?
`context canceled` — отмена родителя автоматически отменяет все дочерние контексты.
![[context WithCancel#^cancel-closes-channel]]

WithCancelCause: два вызова cancel с разными ошибками. Что вернёт context.Cause(ctx)?
?
![[context WithCancel#^cancel-cause-first-wins]]

Что вернёт ctx.Err() и context.Cause(ctx) после cancel с причиной?
?
![[context WithCancel#^cancel-cause-immutable]]

С какой версии Go доступен WithCancelCause?
?
![[context WithCancel#^cancel-cause-version]]

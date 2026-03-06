#flashcards/context/with_deadline

Какой тип принимает WithDeadline — абсолютное или относительное время?
?
![[context WithDeadline#^deadline-absolute]]

Как WithTimeout реализован под капотом?
?
![[context WithDeadline#^deadline-wraps-timeout]]

Что происходит если у родительского контекста дедлайн раньше чем у дочернего?
?
![[context WithDeadline#^deadline-parent-wins]]

Когда оправдано использовать WithDeadline вместо WithTimeout? Приведи конкретный сценарий.
?
![[context WithDeadline#^deadline-propagation-usecase]]

Как паттерн ctx.Deadline() + time.Until помогает избежать бессмысленного старта тяжёлых операций?
?
![[context WithDeadline#^deadline-check-remaining]]

В каком проценте случаев WithTimeout удобнее WithDeadline?
?
![[context WithDeadline#^deadline-vs-timeout-99]]

Что выведет этот код?
```go
parent, cancel := context.WithDeadline(
    context.Background(),
    time.Now().Add(1*time.Second),
)
defer cancel()

child, cancel2 := context.WithDeadline(parent, time.Now().Add(10*time.Second))
defer cancel2()

deadline, _ := child.Deadline()
fmt.Println(time.Until(deadline) < 2*time.Second)
```
?
`true` — ребёнок не может продлить время родителя. Дедлайн child = дедлайн parent (1с), несмотря на переданные 10с.
![[context WithDeadline#^deadline-parent-wins]]

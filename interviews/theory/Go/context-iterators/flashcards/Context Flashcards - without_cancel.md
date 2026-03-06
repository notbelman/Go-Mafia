#flashcards/context/without_cancel

Что делает context.WithoutCancel? Что сохраняется, а что теряется?
?
![[context WithoutCancel#^woc-purpose]]
![[context WithoutCancel#^woc-done-nil]]

Назови типичный use-case для context.WithoutCancel.
?
![[context WithoutCancel#^woc-usecase]]
![[context WithoutCancel#^woc-pattern-why]]

Почему нельзя заменить context.WithoutCancel на context.Background() при логировании после отмены?
?
![[context WithoutCancel#^woc-bg-vs-woc]]

Что вернут child.Done() и child.Err() после вызова cancel() на родителе, если child создан через WithoutCancel?
?
![[context WithoutCancel#^woc-done-nil]]

«Не отдавать контекст в детдом» — что означает это правило?
?
![[context WithoutCancel#^woc-never-background]]
![[context WithoutCancel#^woc-bg-vs-woc]]

Какие реальные баги возникали при нарушении принципа parent-child для контекстов?
?
![[context WithoutCancel#^woc-real-bugs]]

Что выведет этот код?
```go
bg := context.Background()
parent, cancel := context.WithTimeout(bg, time.Second)
child := context.WithoutCancel(parent)
cancel()

fmt.Println(parent.Err())
fmt.Println(child.Err())
fmt.Println(child.Done() == nil)
```
?
`context canceled`, `nil`, `true` — WithoutCancel обрывает отмену, поэтому child.Err() == nil и Done() == nil (канал никогда не закроется).
![[context WithoutCancel#^woc-done-nil]]

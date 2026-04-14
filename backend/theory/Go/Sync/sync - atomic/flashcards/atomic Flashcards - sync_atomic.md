#flashcards/atomic/sync_atomic

Дай определение атомарной операции. Что именно гарантируется?
?
![[sync - atomic#^atomic-definition]]

Перечисли 5 основных методов atomic и что каждый делает.
?
![[sync - atomic#^atomic-methods-table]]

Что возвращает Add? Что возвращает Swap?
?
![[sync - atomic#^atomic-methods-table]]

Как работает CompareAndSwap? При каком условии происходит запись?
?
![[sync - atomic#^atomic-methods-table]]

Какие типовые обёртки появились в Go 1.19? Перечисли все.
?
![[sync - atomic#^atomic-types-go119]]

До Go 1.19 как работали с atomic? Что изменилось в 1.19?
?
![[sync - atomic#^atomic-types-go119]]

Что выведет этот код?
```go
var n atomic.Int32
old := n.Swap(42)
fmt.Println(old, n.Load())
```
?
`0 42` — Swap возвращает старое значение (0), устанавливает новое (42).
![[sync - atomic#^atomic-methods-table]]

Что выведет и почему?
```go
var n atomic.Int64
n.Store(10)
ok := n.CompareAndSwap(10, 20)
fmt.Println(ok, n.Load())
ok2 := n.CompareAndSwap(10, 30)
fmt.Println(ok2, n.Load())
```
?
`true 20` затем `false 20` — первый CAS: текущее==old(10), записывает 20, возвращает true. Второй CAS: текущее==20 != old(10), запись не происходит, возвращает false.
![[sync - atomic#^atomic-methods-table]]

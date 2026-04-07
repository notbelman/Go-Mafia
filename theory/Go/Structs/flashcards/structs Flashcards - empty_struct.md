#flashcards/structs/empty_struct

Чем Go отличается от C++ в отношении размера пустой структуры?
?
![[Пустые_структуры#^empty-size-zero]]

Сколько байт занимает `struct{}` в Go? А в C++?
?
![[Пустые_структуры#^empty-size-zero]]

Что выведет этот код?
```go
var a struct{}
var b struct{}
var c [0]int
fmt.Println(&a == &b, unsafe.Sizeof(a))
_ = c
```
?
`true 0` — все `struct{}` и `[0]T` в одном контексте (горутина/стек) указывают на один фиктивный адрес. Размер — 0 байт.
![[Пустые_структуры#^empty-same-addr]] + ![[Пустые_структуры#^empty-size-zero]]

Гарантирован ли единый адрес для всех `struct{}` в программе глобально?
?
![[Пустые_структуры#^empty-addr-goroutine]]

Почему адрес `struct{}` в разных горутинах может быть разным?
?
![[Пустые_структуры#^empty-addr-goroutine]]

Каков размер этой структуры и почему?
```go
type C struct {
    X int64
    _ struct{}
}
```
?
![[Пустые_структуры#^empty-last-field-padding]]

Каков размер этой структуры и почему?
```go
type A struct {
    _ struct{}
    X int64
}
```
?
![[Пустые_структуры#^empty-last-field-padding]]

Почему Go добавляет padding, когда `struct{}` стоит последним полем?
?
![[Пустые_структуры#^empty-padding-gc-reason]]

Что произойдёт без padding, если взять адрес пустого поля `struct{}` в конце структуры?
?
![[Пустые_структуры#^empty-padding-gc-reason]]

Почему `map[string]struct{}` используют вместо `map[string]bool` для реализации set?
?
![[Пустые_структуры#^empty-set-pattern]]

Что общего у `struct{}` и `[0]int` с точки зрения памяти?
?
![[Пустые_структуры#^empty-same-addr]]

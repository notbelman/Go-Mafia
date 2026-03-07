#flashcards/interfaces/eface

Чем структура eface отличается от iface? Что хранит вместо itab?
?
![[eface#^eface-no-itab]]

Почему для пустого интерфейса используется отдельная структура eface, а не iface?
?
![[eface#^eface-separate-struct]]

Почему type switch по eface быстрее чем по iface?
?
![[eface#^eface-type-direct]]

Назови два преимущества eface над iface
?
![[eface#^eface-advantages]]

Что происходит при упаковке скаляра (int, bool) в any? Есть ли аллокация?
?
![[eface#^eface-boxing-scalar-alloc]]

Что происходит при упаковке указателя в any? Есть ли аллокация?
?
![[eface#^eface-boxing-ptr-no-alloc]]

Что выведет этот код?
```go
var a any = 42
var b any = 0
fmt.Println(a == b)
```
?
`false` — 42 != 0. Сравнение eface сначала сравнивает `_type` (оба int → совпадает), затем значение data. 42 ≠ 0.
![[eface#^eface-separate-struct]]

Что выведет этот код?
```go
x := 42
var a any = x
var b any = x
fmt.Println(a == b)
```
?
`true` — оба eface имеют одинаковый `_type` (int) и одинаковое значение (42). Скаляры сравниваются по значению, не по адресу.
![[eface#^eface-boxing-scalar-alloc]]

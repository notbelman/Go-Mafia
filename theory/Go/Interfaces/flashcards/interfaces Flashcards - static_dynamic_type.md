#flashcards/interfaces/static_dynamic_type

Что такое статический тип переменной-интерфейса? Может ли он измениться?
?
![[Статический и динамический тип#^static-type-def]]

Что такое динамический тип переменной-интерфейса? Когда он меняется?
?
![[Статический и динамический тип#^dynamic-type-def]]

Что проверяет type assertion — статический или динамический тип?
?
![[Статический и динамический тип#^assertion-dynamic]]

Чем отличается type assertion с конкретным типом T от type assertion с интерфейсом T?
?
![[Статический и динамический тип#^assertion-interface-t]]

Что выведет этот код?
```go
type Fooer interface { Foo() }
type Barer interface { Bar() }

type Thing struct{}
func (t Thing) Foo() {}
func (t Thing) Bar() {}

var f Fooer = Thing{}
b, ok := f.(Barer)
fmt.Println(ok)
```
?
`true` — хотя статический тип `f` это `Fooer` (без метода `Bar`), type assertion смотрит на динамический тип `Thing`, у которого `Bar()` есть. `Thing` реализует `Barer`.
![[Статический и динамический тип#^assertion-static-vs-dynamic]]

Как Go сравнивает два интерфейса оператором ==?
?
![[Статический и динамический тип#^iface-eq]]

Что выведет этот код?
```go
var x any = 3
var y any = 3
var z any = 3.0
fmt.Println(x == y)
fmt.Println(x == z)
```
?
`true` затем `false`. Первое сравнение: динамический тип `int` и значение `3` совпадают. Второе: динамические типы разные — `int` vs `float64`, несмотря на то что значения «одинаковые».
![[Статический и динамический тип#^iface-eq-example]]

Для понимания каких трёх механизмов Go критически важно разбираться в статическом и динамическом типах?
?
![[Статический и динамический тип#^static-dynamic-usage]]

Что произойдёт со статическим и динамическим типом после каждого присваивания?
```go
var s Stringer
s = time.Now()
s = myDate{}
```
?
![[Статический и динамический тип#^static-forever-dynamic-changes]]

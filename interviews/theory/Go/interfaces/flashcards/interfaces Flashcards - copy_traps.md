#flashcards/interfaces/copy_traps

Что происходит с оригинальным значением после присваивания в интерфейс? Почему?
?
![[Копирование и ловушки с типами#^copy-on-assign]]

Что выведет этот код?
```go
value := 100
var i any = value
value = 200
fmt.Println(i)
```
?
`100` — при присваивании в интерфейс значение копируется. Изменение `value` не влияет на копию внутри интерфейса.
![[Копирование и ловушки с типами#^copy-on-assign]]

Почему нельзя присвоить значение (не указатель) в интерфейс, если тип реализует его через pointer receiver?
?
![[Копирование и ловушки с типами#^pointer-receiver-rule]]

Что выведет этот код — скомпилируется или нет?
```go
type Obj struct{}
func (o *Obj) Do() {}
type Doer interface{ Do() }

var d Doer = Obj{}
fmt.Println(d)
```
?
Не скомпилируется — `Obj` не реализует `Doer`, потому что `Do()` определён на `*Obj`. Нужно `d = &Obj{}`.
![[Копирование и ловушки с типами#^pointer-receiver-rule]]

Value receiver vs pointer receiver: что можно присвоить в интерфейс в каждом случае?
?
![[Копирование и ловушки с типами#^value-vs-pointer-receiver]]

Почему `[]int` нельзя присвоить в `[]any` напрямую? Как обойти?
?
![[Копирование и ловушки с типами#^slice-to-any-slice]]

Что произойдёт в рантайме при попытке сравнить два `any`, содержащих `[]int`?
?
![[Копирование и ловушки с типами#^uncomparable-panic]]

Что выведет этот код?
```go
m := map[any]struct{}{}
m[[]int{1, 2}] = struct{}{}
fmt.Println("ok")
```
?
Паника в рантайме: `panic: runtime error: hash of unhashable type []int`. Компилятор не видит конкретный тип за `any`, проверка только в рантайме.
![[Копирование и ловушки с типами#^uncomparable-panic]]

Какие конкретные типы являются несравниваемыми и вызовут панику за интерфейсом?
?
![[Копирование и ловушки с типами#^uncomparable-types]]

Почему type assertion `s.(Shape)` может вернуть `ok=false`, даже если тип имеет метод `Area()`?
?
![[Копирование и ловушки с типами#^sig-mismatch-trap]]

Что такое interface guard и от чего он защищает?
?
![[Копирование и ловушки с типами#^interface-guard-benefit]]

Что выведет этот код?
```go
type Shape interface{ Area() float64 }
type Circle struct{ R float64 }
func (c Circle) Area() int { return int(c.R * c.R) }

var s any = Circle{R: 5}
_, ok := s.(Shape)
fmt.Println(ok)
```
?
`false` — у `Circle` метод `Area() int`, а интерфейс требует `Area() float64`. Разная сигнатура — это разные методы, интерфейс не реализован.
![[Копирование и ловушки с типами#^sig-mismatch-trap]]

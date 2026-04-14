#flashcards/structs/methods_receivers

Что такое метод в Go на уровне ассемблера? Что такое ресивер с точки зрения компилятора?
?
![[Методы и ресиверы#^method-is-func]] + ![[Методы и ресиверы#^method-asm-sugar]]

Как вызвать метод явно через тип, минуя синтаксический сахар?
?
![[Методы и ресиверы#^method-explicit-call]]

Что произойдёт при вызове метода на nil-указателе? Когда будет паника, а когда нет?
?
![[Методы и ресиверы#^method-nil-receiver]]

Что выведет этот код?
```go
type Data struct{ Value int }

func (d *Data) Safe() { fmt.Println("ok") }
func (d *Data) Unsafe() { fmt.Println(d.Value) }

var d *Data
d.Safe()
```
?
`ok` — метод вызывается с nil первым аргументом. Паники нет, так как `Safe()` не разыменовывает `d`.
![[Методы и ресиверы#^nil-receiver-example]]

Что произойдёт?
```go
type Data struct{ Value int }
func (d *Data) Unsafe() { fmt.Println(d.Value) }
var d *Data
d.Unsafe()
```
?
`panic: nil pointer dereference` — при обращении к `d.Value` происходит разыменование nil-указателя.
![[Методы и ресиверы#^nil-receiver-example]]

`obj.Print()` → что происходит под капотом если Print объявлен с value receiver?
?
![[Методы и ресиверы#^method-lowering-example]]

`ptr.Print()` → что происходит под капотом если Print объявлен с value receiver?
?
![[Методы и ресиверы#^ptr-auto-deref]]

Можно ли объявить два метода с одним именем `_` для одного типа? Можно ли их вызвать?
?
![[Методы и ресиверы#^method-blank-ident]]

Что выведет этот код?
```go
type Data struct{ Value int }
func (d Data) Print() { fmt.Println(d.Value) }

obj := Data{Value: 42}
Data.Print(obj)
```
?
`42` — вызов через тип — это то как работает `obj.Print()` под капотом. Явный синтаксис с передачей ресивера как аргумента.
![[Методы и ресиверы#^method-lowering-example]]

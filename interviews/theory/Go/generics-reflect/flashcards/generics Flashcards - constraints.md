#flashcards/generics/constraints

Что такое constraint в Go дженериках?
?
![[Constraints#^constraint-def]]

Как записать OR-условие в constraint? Как AND?
?
![[Constraints#^constraint-or-and]]

Что означает `~int` в constraint? Чем отличается от `int`?
?
![[Constraints#^constraint-tilde]]

Какие два зарезервированных constraint есть в Go и что они означают?
?
![[Constraints#^constraint-reserved]]

Где живут `constraints.Ordered`, `constraints.Integer`, `constraints.Float`? Почему не в стандартной библиотеке?
?
![[Constraints#^constraint-experimental]]

Тип должен быть int или float64, И иметь метод String(). Как написать такой constraint?
?
![[Constraints#^constraint-and-rule]]

Есть `type MyInt int`. Почему `process1[T Integer1]` не примет `MyInt`, а `process2[T Integer2]` примет?
?
![[Constraints#^constraint-tilde-example]]

Покажи три способа записать inline constraint — именованный, `interface{}`, и сокращённый.
?
![[Constraints#^constraint-inline]]

Почему для ключей map нужен constraint `comparable`, а не `any`?
?
![[Constraints#^constraint-comparable]]

Что выведет этот код?
```go
type MyInt int

type C interface{ ~int }

func double[T C](v T) T { return v * 2 }

fmt.Println(double(MyInt(5)))
```
?
`10` — `MyInt` имеет базовый тип `int`, поэтому удовлетворяет `~int`. Компилятор инстанцирует `double[MyInt]`.
![[Constraints#^constraint-tilde-example]]

Что выведет этот код?
```go
type C interface{ int | string }

func printType[T C](v T) {
    fmt.Printf("%T\n", v)
}

printType(42)
printType("hello")
```
?
`int` и `string` — тип выводится из аргумента. `%T` печатает конкретный тип, потому что `T` инстанцируется в конкретный тип при вызове.
![[Constraints#^constraint-or-rule]]

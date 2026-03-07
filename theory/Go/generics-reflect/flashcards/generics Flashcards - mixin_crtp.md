#flashcards/generics/mixin_crtp

Что такое Mixin-паттерн в контексте Go дженериков?
?
![[Mixin и CRTP#^mixin-def]]

Как называется C++ паттерн, аналогом которого является Go Mixin с дженериками?
?
![[Mixin и CRTP#^mixin-crtp-analogy]]

Какую проблему решает Mixin с дженериками? Почему без дженериков это не решить чисто?
?
![[Mixin и CRTP#^mixin-problem]]

Что вернёт Clone() без дженериков, если встроить обычную структуру `Clonable`? Чем это плохо?
?
![[Mixin и CRTP#^mixin-without-generics]]

Почему `Data1{Clonable[Data1]}` называется "передаёт себя как параметр типа"? Что происходит при инстанцировании?
?
![[Mixin и CRTP#^mixin-instantiation]]

Что вернёт `d1.Clone()` если `d1` имеет тип `Data1`, встраивающий `Clonable[Data1]`?
?
![[Mixin и CRTP#^mixin-solution]]

Есть ли в Go Mixin/CRTP ограничение по сравнению с C++ CRTP? Что именно нельзя сделать?
?
![[Mixin и CRTP#^mixin-cpp-comparison]]

Что выведет этот код?
```go
type Clonable[T any] struct{}

func (c Clonable[T]) Clone() *T {
    return new(T)
}

type Foo struct {
    Clonable[Foo]
    X int
}

f := Foo{X: 99}
clone := f.Clone()
fmt.Printf("%T, X=%d\n", clone, clone.X)
```
?
`*main.Foo, X=0` — Clone возвращает `*Foo` (правильный тип), но `new(T)` создаёт нулевое значение. Поля не копируются, `X` равен нулю.
![[Mixin и CRTP#^mixin-solution]]

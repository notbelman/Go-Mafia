#flashcards/generics/factory_decorator

Как устроена обобщённая фабрика через Creator[T]? Что такое Creator и как работает NewInstance?
?
![[Обобщённая фабрика и декоратор#^factory-creator-pattern]]

Что выведет этот код?
```go
type Creator[T any] func() T

func NewInstance[T any](c Creator[T]) T {
    return c()
}

type Book struct{ Title string }

var bc Creator[Book] = func() Book { return Book{Title: "Go"} }
b := NewInstance(bc)
fmt.Println(b.Title)
```
?
`Go` — `NewInstance` вызывает переданный Creator, возвращает `Book{Title: "Go"}`. Тип выводится из типа Creator.
![[Обобщённая фабрика и декоратор#^factory-creator-pattern]]

Как устроен обобщённый декоратор? В каком порядке применяются функции?
?
![[Обобщённая фабрика и декоратор#^decorator-pattern-order]]

Что выведет этот код?
```go
func decorate[T any](fn func(T), decorator func(T) T) func(T) {
    return func(input T) {
        fn(decorator(input))
    }
}
func printInt(v int) { fmt.Println(v) }
func triple(v int) int { return v * 3 }

decorated := decorate(printInt, triple)
decorated(5)
```
?
`15` — сначала `triple(5) = 15`, потом `printInt(15)`. Порядок: decorator применяется первым, результат передаётся в fn.
![[Обобщённая фабрика и декоратор#^decorator-pattern-order]]

Какую проблему решают типизированные unsafe-обёртки с дженериками? Что было до и что стало после?
?
![[Обобщённая фабрика и декоратор#^unsafe-wrapper-generics]]

Какой пример из stdlib демонстрирует типизированную обёртку над unsafe с дженериками?
?
![[Обобщённая фабрика и декоратор#^unsafe-wrapper-generics]]

Почему паттерны из ООП и ФП (фабрика, декоратор) усиливаются дженериками, а не просто дублируются для каждого типа?
?
![[Обобщённая фабрика и декоратор#^factory-creator-pattern]] + ![[Обобщённая фабрика и декоратор#^decorator-pattern-order]]

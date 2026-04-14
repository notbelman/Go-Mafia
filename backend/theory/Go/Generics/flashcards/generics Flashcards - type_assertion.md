#flashcards/generics/type_assertion

Почему нельзя делать `v.(type)` напрямую над generic-параметром T?
?
![[Type assertion в дженериках#^ta-why-forbidden]]

Как правильно сделать type switch внутри generic-функции?
?
![[Type assertion в дженериках#^ta-any-cast]]

В какой момент определяется конкретный тип generic-параметра T — compile-time или runtime? Почему это парадокс?
?
![[Type assertion в дженериках#^ta-paradox]]

Что такое proposal `switch type T`? Почему его нет в текущем Go?
?
![[Type assertion в дженериках#^ta-proposal-detail]]

Что произойдёт при компиляции?
```go
func process[T any](v T) string {
    switch v.(type) {
    case int:
        return "int"
    default:
        return "other"
    }
}
```
?
Ошибка компиляции — `T` не является интерфейсом, поэтому type assertion над ним запрещён. Нужно привести к `any(v).(type)`.
![[Type assertion в дженериках#^ta-why-forbidden]]

Что выведет этот код?
```go
func describe[T int | string](v T) string {
    switch any(v).(type) {
    case int:
        return "int"
    case string:
        return "string"
    }
    return "unknown"
}

fmt.Println(describe(42))
fmt.Println(describe("hi"))
```
?
`int` и `string` — `any(v)` оборачивает значение в пустой интерфейс, type switch срабатывает в рантайме. Несмотря на то что constraint явно указывает `int | string`, ветвление всё равно происходит через рантайм-интерфейс.
![[Type assertion в дженериках#^ta-solution]]

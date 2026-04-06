#flashcards/functions/decorator_composition

Что такое декоратор в функциональном стиле? Какую проблему решает?
?
![[Декоратор и композиция#^decorator-what]]

Что такое композиция функций (pipeline)? Как работает порядок применения?
?
![[Декоратор и композиция#^composition-what]]

Что объединяет декоратор и композицию с точки зрения теории функций?
?
Оба являются функциями высшего порядка (ФВП) — принимают и/или возвращают функции.
![[Декоратор и композиция#^decorator-what]]

Что такое continuation (callback) паттерн? Когда использовать и какой риск?
?
![[Декоратор и композиция#^continuation-what]]

Можно ли передать декоратору анонимную функцию inline, без присваивания переменной?
?
![[Декоратор и композиция#^decorator-inline]]

Что выведет этот код?
```go
func compose(fns ...func(int) int) func(int) int {
    return func(val int) int {
        for _, fn := range fns {
            val = fn(val)
        }
        return val
    }
}

func main() {
    square := func(x int) int { return x * x }
    negate := func(x int) int { return -x }
    transform := compose(square, negate, square)
    fmt.Println(transform(4))
}
```
?
`256`. Pipeline: 4 → square → 16 → negate → -16 → square → 256. Функции применяются слева направо в порядке передачи в compose.
![[Декоратор и композиция#^composition-what]]

Что выведет этот код?
```go
type BinOp func(int, int) int

func withLogging(op BinOp) BinOp {
    return func(a, b int) int {
        fmt.Printf("calling with %d, %d\n", a, b)
        result := op(a, b)
        fmt.Printf("result: %d\n", result)
        return result
    }
}

func main() {
    add := func(a, b int) int { return a + b }
    loggedAdd := withLogging(add)
    loggedAdd(3, 5)
}
```
?
```
calling with 3, 5
result: 8
```
`withLogging` — классический декоратор: оборачивает BinOp в новую BinOp, добавляя логирование до и после вызова. Бизнес-логика `add` не изменена.
![[Декоратор и композиция#^decorator-what]]

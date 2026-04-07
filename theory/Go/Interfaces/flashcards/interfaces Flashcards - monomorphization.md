#flashcards/interfaces/monomorphization

Что такое мономорфизация в контексте Go generics?
?
![[Мономорфизация#^mono-definition]]

Сколько функций создаст компилятор для `func Sum[T int | float64](a, b T) T`?
?
![[Мономорфизация#^mono-example]]

Почему generics с мономорфизацией не используют itab и indirect call?
?
![[Мономорфизация#^mono-no-itab]]

Как скорость вызова generic-функции соотносится со статической диспетчеризацией?
?
![[Мономорфизация#^mono-perf-equiv]]

Что выведет этот код и чем отличается от вызова через интерфейс?
```go
func Sum[T int | float64](a, b T) T { return a + b }

func main() {
    fmt.Println(Sum(1, 2))
    fmt.Println(Sum(1.5, 2.5))
}
```
?
`3` и `4` — компилятор создал отдельные `Sum_int` и `Sum_float64`. Прямые вызовы без itab и без overhead интерфейса.
![[Мономорфизация#^mono-example]]

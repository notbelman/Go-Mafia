#flashcards/functions/hof

Что такое функция высшего порядка (ФВП)? Дай определение.
?
![[ФВП и предикаты#^hof-def]]

Что такое предикат в контексте функционального программирования?
?
![[ФВП и предикаты#^predicate-def]]

Зачем использовать предикат в сортировке? Какую проблему он решает?
?
![[ФВП и предикаты#^predicate-generalize]]

Что пришлось бы делать без предиката для сортировки по возрастанию и убыванию?
?
![[ФВП и предикаты#^predicate-without]]

Назови 4 паттерна ФВП, которые распространены в Go.
?
![[ФВП и предикаты#^hof-patterns]]

В примере с `forEach` — является ли она ФВП? Почему?
?
![[ФВП и предикаты#^foreach-hof]]

В примере с `multiplier` — является ли она ФВП? Почему? Какой паттерн она реализует?
?
![[ФВП и предикаты#^hof-return-func]]

Что выведет этот код?
```go
func multiplier(n int) func(int) int {
    return func(x int) int { return x * n }
}

func main() {
    double := multiplier(2)
    triple := multiplier(3)
    fmt.Println(double(5), triple(5))
}
```
?
`10 15` — `multiplier` возвращает замыкание, захватывающее `n`. `double` захватывает `n=2`, `triple` — `n=3`. Каждый вызов `multiplier` создаёт независимую функцию.
![[ФВП и предикаты#^hof-return-func]]

Что выведет этот код?
```go
func forEach(arr []int, fn func(int) int) []int {
    result := make([]int, len(arr))
    for i, v := range arr {
        result[i] = fn(v)
    }
    return result
}

func main() {
    src := []int{1, 2, 3}
    dst := forEach(src, func(x int) int { return x * x })
    src[0] = 99
    fmt.Println(dst[0])
}
```
?
`1` — `forEach` создаёт новый `result` через `make`, не разделяет backing array с `src`. Изменение `src[0]` не влияет на `dst`.
![[ФВП и предикаты#^foreach-hof]]

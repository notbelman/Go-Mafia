#flashcards/functions/anon_variadic

В чём разница между двумя способами использования анонимной функции: присвоить переменной vs вызвать сразу?
?
![[Анонимные функции и variadic#^anon-two-styles]]

Что такое variadic параметр на уровне типов внутри функции?
?
![[Анонимные функции и variadic#^variadic-sugar]]

Какие ограничения на variadic параметр в Go?
?
![[Анонимные функции и variadic#^variadic-constraints]]

Что выведет этот код?
```go
func process(ptrs ...*int) {
    fmt.Println(ptrs)
}

func main() {
    process(nil)
    process(nil...)
}
```
?
`[<nil>]` на первой строке, `[]` на второй. `process(nil)` создаёт slice `[]*int{nil}` — один элемент nil-указатель. `process(nil...)` распаковывает nil slice — передаётся пустой slice.
![[Анонимные функции и variadic#^variadic-nil]]

Что выведет этот код?
```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

func main() {
    s := []int{1, 2, 3}
    fmt.Println(sum(s...))
    fmt.Println(sum())
}
```
?
`6`, затем `0`. `sum(s...)` распаковывает slice в variadic — передаются элементы 1,2,3. `sum()` — nums будет пустым `[]int{}`, цикл не выполняется.
![[Анонимные функции и variadic#^variadic-sugar]]

Что выведет этот код?
```go
funcs := make([]func(), 3)
for i := 0; i < 3; i++ {
    i := i  // shadowing
    funcs[i] = func() { fmt.Println(i) }
}
for _, f := range funcs {
    f()
}
```
?
`0`, `1`, `2`. Благодаря `i := i` (shadowing) каждая анонимная функция захватывает свою копию `i`. Без shadowing вывело бы `3 3 3`.
![[Анонимные функции и variadic#^anon-two-styles]]

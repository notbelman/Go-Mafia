#flashcards/functions/inlining

Что такое inlining и какой overhead он убирает?
?
![[Inlining функций#^inlining-what]]

Как работает budget-модель inlining? Что такое cost функции?
?
![[Inlining функций#^inlining-budget-formula]]

Какой порог бюджета для inlining в Go?
?
![[Inlining функций#^inlining-budget-threshold]]

Стабилен ли бюджет inlining между версиями Go?
?
![[Inlining функций#^inlining-budget-unstable]]

Как явно запретить inlining конкретной функции?
?
![[Inlining функций#^inlining-noinline]]

Почему компилятор не инлайнит всё подряд? Назови трейдофф.
?
![[Inlining функций#^inlining-tradeoff]]

Назови две причины писать маленькие компактные функции в Go.
?
![[Inlining функций#^inlining-small-funcs]]

Что выведет этот код и почему?
```go
//go:noinline
func add(a, b int) int { return a + b }

func main() {
    x := add(3, 5)
    _ = x
}
```
Изменится ли поведение если убрать директиву?
?
Поведение не изменится — результат тот же `8`. Но без `//go:noinline` компилятор может встроить тело `add` прямо в main, убрав overhead CALL/RET. С директивой — всегда будет настоящий вызов.
![[Inlining функций#^inlining-noinline]]

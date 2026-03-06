#flashcards/functions/defer_named_returns

Может ли defer изменить возвращаемое значение функции? При каком условии?
?
![[Defer и именованные возвращаемые#^dnr-can-modify]]
<!--SR:!2026-02-27,3,250-->

Что выведет этот код?
```go
func calculate(value int) (result int) {
    defer func() { result += value }()
    return value + value
}
fmt.Println(calculate(5))
```
?
`15` — return записывает 10 в именованную переменную result, затем выполняется defer, который добавляет value (5) к той же переменной result. Возвращается итоговое 15.
![[Defer и именованные возвращаемые#^dnr-named-example]]
<!--SR:!2026-02-27,3,250-->

Что выведет этот код?
```go
func calculate(value int) int {
    result := value + value
    defer func() { result += value }()
    return result
}
fmt.Println(calculate(5))
```
?
`10` — return копирует значение локальной переменной result (10) в анонимную возвращаемую переменную. Defer модифицирует локальный result, но возвращаемая копия уже зафиксирована на значении 10.
![[Defer и именованные возвращаемые#^dnr-local-example]]
<!--SR:!2026-02-28,4,270-->

Опиши точный порядок выполнения при return — 4 шага.
?
![[Defer и именованные возвращаемые#^dnr-return-order]]
<!--SR:!2026-02-27,3,250-->

Почему defer может изменить именованное возвращаемое, но не локальную переменную?
?
![[Defer и именованные возвращаемые#^dnr-return-order]]
![[Defer и именованные возвращаемые#^dnr-local-copy]]

Что такое defer inlining в Go 1.14+? В чём суть оптимизации?
?
![[Defer и именованные возвращаемые#^dnr-inlining-detail]]
<!--SR:!2026-02-27,3,250-->

При каких условиях компилятор Go 1.14+ применяет defer inlining? Назови ограничения.
?
![[Defer и именованные возвращаемые#^dnr-inlining-detail]]
<!--SR:!2026-02-28,4,270-->

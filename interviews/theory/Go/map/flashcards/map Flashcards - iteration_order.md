#flashcards/map/iteration_order

Гарантирован ли порядок итерации по Go map? Что будет при двух последовательных range по одной и той же map?
?
![[iteration order#^iter-random]]

Как именно Go рандомизирует начало итерации? Опиши механизм по шагам.
?
![[iteration order#^iter-mechanism]]

Почему в Go 1.0 порядок итерации был детерминированным, и к чему это привело?
?
![[iteration order#^iter-history]]

С какой версии Go рандомизировал порядок итерации map?
?
![[iteration order#^iter-since-go13]]

Как получить отсортированный обход map? Напиши паттерн.
?
![[iteration order#^iter-sorted-pattern]]

Что выведет этот код?
```go
m := map[string]int{"a": 1, "b": 2, "c": 3}
for k := range m { fmt.Print(k, " ") }
fmt.Println()
for k := range m { fmt.Print(k, " ") }
```
?
Порядок непредсказуем и может быть разным в двух вызовах — например `b a c` и `c b a`. Go рандомизирует стартовый бакет и offset при каждом вызове range.
![[iteration order#^iter-random]]

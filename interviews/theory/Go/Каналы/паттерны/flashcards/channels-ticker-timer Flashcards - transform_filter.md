#flashcards/channels-patterns/transform_filter

Что такое Transform в контексте паттернов каналов? Чем отличается от обычной функции?
?
![[Transform и Filter#^transform-def]]

Что такое Filter? В чём его ключевое отличие от Transform?
?
![[Transform и Filter#^filter-def]]

Что выведет этот код?
```go
result := filter(
    transform(
        gen(1, 2, 3, 4, 5),
        func(x int) int { return x * x },
    ),
    func(x int) bool { return x > 10 },
)
for v := range result { fmt.Println(v) }
```
?
`16` и `25` — сначала все числа возводятся в квадрат (1,4,9,16,25), затем фильтр пропускает только >10. Каждая стадия — отдельная горутина, данные текут последовательно через каналы.
![[Transform и Filter#^transform-filter-compose]]

Почему Transform и Filter называют декораторами? Что это значит в терминах сигнатуры?
?
![[Transform и Filter#^transform-filter-compose]]

Опиши общий шаблон всех функций-декораторов каналов (Transform, Filter, Fan-In, Fan-Out, Tee).
?
![[Transform и Filter#^transform-filter-pattern]]

Перечисли практические применения Filter.
?
![[Transform и Filter#^filter-usecases]]

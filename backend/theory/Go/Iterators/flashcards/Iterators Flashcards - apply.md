#flashcards/Iterators/apply

Какую проблему решают итераторы для пользовательских структур данных (деревья, графы)?
?
![[Итераторы_применение#^apply-why]]
![[Итераторы_применение#^apply-off-by-one-fix]]

По каким типам range работал до Go 1.23? Что нельзя было обойти напрямую?
?
![[Итераторы_применение#^apply-before-123]]

Перечисли stdlib-функции для итерации по срезам (пакет slices): названия и возвращаемые типы.
?
![[Итераторы_применение#^apply-slices-types]]

Перечисли stdlib-функции для итерации по map (пакет maps): названия и возвращаемые типы.
?
![[Итераторы_применение#^apply-maps-types]]

Почему maps.Values и slices.Values возвращают итератор, а не срез? В чём преимущество?
?
![[Итераторы_применение#^apply-lazy-eval]]

Как строится pipeline из итераторов? Какую сигнатуру имеют функции-трансформеры?
?
![[Итераторы_применение#^apply-pipeline-pattern]]

Какое преимущество pipeline из итераторов перед pipeline из срезов с точки зрения памяти?
?
![[Итераторы_применение#^apply-pipeline-no-alloc]]

Каков overhead итераторов по сравнению с обычным range? Есть ли смысл избегать итераторов из соображений производительности?
?
![[Итераторы_применение#^apply-perf-conclusion]]

Что возвращает slices.Backward — iter.Seq или iter.Seq2? Какие значения отдаёт?
?
![[Итераторы_применение#^apply-slices-types]]

Что выведет этот код?
```go
s := []string{"a", "b", "c"}
for i, v := range slices.Backward(s) {
    fmt.Printf("%d:%s ", i, v)
}
```
?
`2:c 1:b 0:a ` — slices.Backward обходит срез с конца, возвращая оригинальные индексы (не 0,1,2 а 2,1,0). Тип — iter.Seq2[int, T].
![[Итераторы_применение#^apply-slices-types]]

Что выведет этот код?
```go
source := slices.Values([]int{1, 2, 3, 4, 5})

filtered := func(yield func(int) bool) {
    for v := range source {
        if v%2 == 0 {
            if !yield(v) { return }
        }
    }
}

for v := range filtered {
    fmt.Print(v, " ")
}
```
?
`2 4 ` — Filter-итератор оборачивает source, пропуская нечётные. Промежуточный срез не создаётся — ленивые вычисления.
![[Итераторы_применение#^apply-pipeline-pattern]]
![[Итераторы_применение#^apply-pipeline-no-alloc]]

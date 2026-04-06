#flashcards/Iterators/internals

С какой версии Go появилась поддержка range по пользовательским функциям?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-go123]]

Что такое yield в контексте итераторов Go? Что он принимает и что возвращает?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-yield-def]]

Что произойдёт если внутри range вызвать break, а итератор не проверяет результат yield?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-panic-message]]

Почему runtime Go паникует при непроверенном yield после break? Какова причина этой защиты?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-yield-panic-rule]]

Что лежит под капотом итераторов Go? Почему стек не нужен?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-under-hood]]

Чем отличаются iter.Seq[V] и iter.Seq2[K,V]? Каким встроенным range-типам они аналогичны?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-seq-one]]
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-seq2-two]]

Каков базовый тип iter.Seq и iter.Seq2? Это интерфейсы или что-то другое?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-types-def]]

Опиши паттерн простого итератора: какую структуру имеет функция-итератор и когда нужно возвращаться?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-simple-pattern]]

Как break внутри range передаёт сигнал остановки в итератор?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-yield-bool-break]]

Зачем параметры итератора передаются через замыкание, а не через yield?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-closure-pattern]]
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-closure-capture]]

yield — это точка переключения. Опиши порядок выполнения: что происходит когда итератор вызывает yield?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-yield-switch-point]]

Итераторы реализованы через stackless coroutines. Что это означает в терминах реализации?
?
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-coroutine-stackless]]
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-fsm-future]]

Что выведет этот код?
```go
func StopAt(n int) iter.Seq[int] {
    return func(yield func(int) bool) {
        for i := 0; i < 10; i++ {
            if !yield(i) {
                return
            }
        }
    }
}

for v := range StopAt(10) {
    if v == 3 {
        break
    }
    fmt.Println(v)
}
```
?
`0`, `1`, `2` — цикл останавливается на break при v==3. yield вернёт false, итератор корректно завершится через return. Паники нет, потому что результат yield проверяется.
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-yield-bool-break]]

Что выведет этот код?
```go
func Bad(yield func(int) bool) {
    for i := 0; i < 5; i++ {
        yield(i) // не проверяем bool
    }
}

for v := range Bad {
    if v == 2 {
        break
    }
    fmt.Println(v)
}
```
?
Паника: `range function continued after loop body exit`. После break range посылает false через yield, но итератор продолжает цикл — runtime обнаруживает нарушение и паникует.
![[WORK-BASE/interviews/theory/Go/context-iterators/Итераторы#^iter-panic-message]]

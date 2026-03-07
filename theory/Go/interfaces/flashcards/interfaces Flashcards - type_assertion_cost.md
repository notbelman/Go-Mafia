#flashcards/interfaces/type_assertion_cost

Почему assertion к конкретному типу `x.(T)` — O(1)?
?
![[Стоимость type assertion#^ta-concrete-o1]]

Какой механизм под капотом при `x.(int)`?
?
![[Стоимость type assertion#^ta-concrete-impl]]

Почему assertion к интерфейсу `x.(I)` дороже, чем к конкретному типу?
?
![[Стоимость type assertion#^ta-iface-mechanism]]

Что происходит при первом вызове assertion к интерфейсу?
?
![[Стоимость type assertion#^ta-iface-first-call]]

Что происходит при повторном assertion к тому же интерфейсу?
?
![[Стоимость type assertion#^ta-iface-repeated]]

Какая сложность вычисления itab при первом assertion к интерфейсу — и почему именно O(n+m)?
?
![[Стоимость type assertion#^ta-iface-mechanism]]

Что выведет этот код и почему второй assertion быстрее первого?
```go
var x any = os.Stdout
r1, ok1 := x.(io.Reader)
r2, ok2 := x.(io.Reader)
fmt.Println(ok1, ok2)
```
?
`true true` — оба успешны. Первый assertion вычисляет itab для пары (io.Reader, *os.File) и кэширует. Второй — O(1) lookup из кэша без пересчёта.
![[Стоимость type assertion#^ta-iface-first-call]]

Как type switch кэширует результаты и зачем это важно в цикле?
?
![[Стоимость type assertion#^ta-switch-cache]]

Что выведет этот код?
```go
var x any = 42
switch x.(type) {
case int:
    fmt.Println("int")
case io.Reader:
    fmt.Println("reader")
}
```
?
`int` — case int проверяется первым через hash-сравнение O(1). До case io.Reader не доходит.
![[Стоимость type assertion#^ta-concrete-impl]]

Когда выбирать assertion к конкретному типу, а когда к интерфейсу?
?
![[Стоимость type assertion#^ta-concrete-o1]]
![[Стоимость type assertion#^ta-iface-expensive]]

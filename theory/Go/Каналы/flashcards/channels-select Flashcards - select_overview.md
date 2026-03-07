#flashcards/channels-select/select_overview

Для чего предназначен оператор `select` в Go?
?
![[select#^select-overview]]

Какие три паттерна `select` компилятор Go оптимизирует в compiler intrinsics? Назови каждый и во что он превращается.
?
![[select#^select-intrinsics-table]]

`select` с одним `case` без `default` — во что оптимизирует компилятор?
?
![[select#^select-intrinsics-table]]

`select` с одним `case send` + `default` — во что оптимизирует компилятор?
?
![[select#^select-intrinsics-table]]

`select` с одним `case recv` + `default` — во что оптимизирует компилятор?
?
![[select#^select-intrinsics-table]]

Зачем нужны compiler intrinsics для `select`? Что они дают?
?
![[select#^select-intrinsics-benefit]]

Что выведет этот код?
```go
ch := make(chan int, 1)
ch <- 42
select {
case v := <-ch:
    fmt.Println("got", v)
default:
    fmt.Println("default")
}
```
?
`got 42` — канал буферизован и содержит значение, поэтому `case` готов сразу. Компилятор оптимизирует этот `select` (один case + default) в `selectnbrecv`, без накладных расходов `selectgo`.
![[select#^select-intrinsics-table]]

#flashcards/channels-select/select_behavior

Что произойдёт в `select`, если готов ровно один канал?
?
![[select - Таблица поведения#^select-behavior-table]]

Что произойдёт в `select`, если одновременно готовы несколько каналов?
?
![[select - Таблица поведения#^select-behavior-table]]

Что произойдёт в `select`, если ни один канал не готов и нет `default`?
?
![[select - Таблица поведения#^select-behavior-table]]

Что произойдёт в `select`, если ни один канал не готов, но есть `default`?
?
![[select - Таблица поведения#^select-behavior-table]]

Что произойдёт в `select`, если все каналы nil и нет `default`?
?
![[select - Таблица поведения#^select-behavior-table]]

Что произойдёт при выполнении `select {}`?
?
![[select - Таблица поведения#^select-behavior-table]]

Что выведет этот код?
```go
var ch chan int // nil channel
select {
case v := <-ch:
    fmt.Println(v)
default:
    fmt.Println("default")
}
```
?
`default` — nil-канал никогда не готов. Но `default` присутствует, поэтому deadlock не случается, выполняется `default`.
![[select - Таблица поведения#^select-behavior-table]]

Что выведет этот код?
```go
var ch chan int // nil channel
select {
case v := <-ch:
    fmt.Println(v)
}
```
?
Deadlock — `fatal error: all goroutines are asleep - deadlock!`. Nil-канал никогда не готов, `default` нет, горутина блокируется навсегда.
![[select - Таблица поведения#^select-behavior-table]]

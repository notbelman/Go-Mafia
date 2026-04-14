#flashcards/channels-nil/nil_channel

Что такое nil-канал в Go и зачем он используется?
?
![[Nil-канал#^nil-chan-definition]]

Почему чтение из nil-канала блокируется навсегда? Опиши механизм.
?
![[Механика операций#^nil-recv-mechanism]]

Почему запись в nil-канал блокируется навсегда?
?
![[Механика операций#^nil-send-mechanism]]

Что произойдёт при `close(nil_channel)`?
?
![[Механика операций#^nil-close-panic]]

Что выделяется в памяти для nil-канала? Существует ли `hchan`?
?
![[Nil-канал#^nil-chan-no-hchan]]

Что происходит с case nil-канала в `select`?
?
![[nil-каналы/Поведение в select#^nil-select-main-feature]]

`select` содержит два case: один с nil-каналом, другой с готовым не-nil каналом. Что произойдёт?
?
![[nil-каналы/Поведение в select#^nil-select-table]]

Все каналы в `select` равны `nil`, `default` нет. Что произойдёт?
?
![[nil-каналы/Поведение в select#^nil-select-all-nil-deadlock]]

Все каналы в `select` равны `nil`, есть `default`. Что выполнится?
?
![[nil-каналы/Поведение в select#^nil-select-table]]

В чём суть паттерна "disable case"? Когда и зачем применяется?
?
![[Disable case#^disable-case-pattern]]

Почему нельзя просто оставить закрытый канал в `select` без обнуления в `nil`?
?
![[Disable case#^disable-case-why-nil]]

Каково условие выхода из цикла в паттерне disable case? Почему именно такое?
?
![[Disable case#^disable-case-loop-condition]]

Что выведет этот код?
```go
var ch chan int
go func() {
    ch <- 42
}()
time.Sleep(time.Second)
fmt.Println("done")
```
?
Программа зависнет — goroutine заблокируется навсегда на записи в nil-канал (`gopark` без `goready`). `"done"` никогда не выведется, eventually deadlock panic от рантайма.
![[Механика операций#^nil-send-mechanism]]

Что выведет этот код?
```go
var ch1, ch2 chan int
ch2 = make(chan int, 1)
ch2 <- 99

select {
case v := <-ch1:
    fmt.Println("ch1:", v)
case v := <-ch2:
    fmt.Println("ch2:", v)
}
```
?
`ch2: 99` — case с `ch1` (nil-канал) игнорируется, выбирается единственный готовый не-nil case.
![[nil-каналы/Поведение в select#^nil-select-main-feature]]

Что выведет этот код?
```go
ch1 := make(chan int)
ch2 := make(chan int)
close(ch1)
close(ch2)

for ch1 != nil || ch2 != nil {
    select {
    case _, ok := <-ch1:
        if !ok { ch1 = nil }
    case _, ok := <-ch2:
        if !ok { ch2 = nil }
    }
}
fmt.Println("both closed")
```
?
`both closed` — паттерн disable case. После того как оба канала закрыты и обнулены, условие цикла становится ложным. Без обнуления был бы busy loop.
![[Disable case#^disable-case-code]]

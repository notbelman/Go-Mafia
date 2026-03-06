#flashcards/channels-other/sendq_example

Опиши по шагам что происходит когда G1 делает `ch <- 10` на пустом небуферизированном канале (никто не читает). Какой sudog создаётся?
?
![[waitq и sudog — пример (sendq)#^sendq-example-full]]

Когда G3 делает `<-ch` и в sendq есть ждущий sender — как передаются данные? Direct Copy или через буфер?
?
![[waitq и sudog — пример (sendq)#^sendq-direct-copy]]

В каком порядке разблокируются горутины из sendq — FIFO или LIFO?
?
![[waitq и sudog — пример (sendq)#^sendq-fifo]]

Что выведет этот код?
```go
ch := make(chan int)
go func() { ch <- 10 }()
go func() { ch <- 20 }()
time.Sleep(10 * time.Millisecond)
fmt.Println(<-ch)
fmt.Println(<-ch)
```
?
`10` и `20` (в этом порядке) — sendq работает как FIFO, первый заблокировавшийся sender разблокируется первым. Данные копируются напрямую из стека sender.
![[waitq и sudog — пример (sendq)#^sendq-fifo]]
![[waitq и sudog — пример (sendq)#^sendq-direct-copy]]

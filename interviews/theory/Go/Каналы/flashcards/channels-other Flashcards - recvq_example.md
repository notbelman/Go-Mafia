#flashcards/channels-other/recvq_example

Опиши по шагам что происходит когда G1 делает `x := <-ch` на пустом канале (никто не отправляет). Что хранится в `sudog.elem`?
?
![[waitq и sudog — пример (recvq)#^recvq-example-full]]

Когда G2 делает `ch <- 10` и в recvq есть ждущий receiver — куда записываются данные? Почему это Direct Copy?
?
![[waitq и sudog — пример (recvq)#^recvq-direct-copy]]

Что выведет этот код?
```go
ch := make(chan int)
var x int
go func() {
    x = 99      // устанавливаем x до отправки
    ch <- x
}()
v := <-ch
fmt.Println(v)
```
?
`99` — sender копирует значение `x` (99) в канал, receiver получает копию. Изменения `x` после send не влияют на полученное значение.
![[waitq и sudog — пример (recvq)#^recvq-direct-copy]]

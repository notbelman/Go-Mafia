#flashcards/channels-non-buf/send

Что произойдёт при send в nil-канал?
?
![[Send (ch - v)#^send-nil]]

Что произойдёт при send в закрытый канал?
?
![[Send (ch - v)#^send-closed-panic]]

Опиши механизм "Direct Copy" при send — пошагово что происходит когда receiver уже ждёт в recvq.
?
![[Send (ch - v)#^send-direct-copy]]

Данные при Direct Copy send проходят через буфер канала?
?
![[Send (ch - v)#^send-direct-copy]]

Опиши механизм send когда recvq пуст — что создаётся, куда встаём, что происходит дальше?
?
![[Send (ch - v)#^send-sleep]]

Что хранится в поле `elem` sudog при send?
?
![[Send (ch - v)#^send-sleep]]

В каком порядке send проверяет условия перед тем как принять решение?
?
![[Send (ch - v)#^send-order]]

В чём асимметрия поведения send и receive для закрытого канала? Почему это важно для паттернов с закрытием?
?
![[Send (ch - v)#^send-vs-recv-closed]]

Что выведет этот код?
```go
ch := make(chan int)
close(ch)
ch <- 1
```
?
`panic: send on closed channel` — send в закрытый канал всегда паникует, в отличие от receive который возвращает zero value.
![[Send (ch - v)#^send-closed-panic]]

Что выведет этот код?
```go
ch := make(chan int)
go func() {
    v := <-ch
    fmt.Println("got", v)
}()
time.Sleep(time.Millisecond)
ch <- 99
fmt.Println("sent")
```
?
```
got 99
sent
```
Горутина ждёт в recvq. Send видит receiver в recvq → Direct Copy: 99 записывается прямо в стек горутины, goready пробуждает её. Порядок вывода: горутина может напечатать "got 99" до или после "sent" — зависит от планировщика, но оба варианта возможны.
![[Send (ch - v)#^send-direct-copy]]

Что выведет этот код?
```go
ch := make(chan int)
go func() {
    ch <- 42
    fmt.Println("send done")
}()
time.Sleep(time.Millisecond * 100)
fmt.Println("before recv")
<-ch
fmt.Println("after recv")
```
?
```
before recv
send done
after recv
```
Горутина заблокирована в sendq. После receive: Direct Copy пробуждает горутину через goready, "send done" выводится после разблокировки. "after recv" — после завершения receive в main.
![[Send (ch - v)#^send-sleep]]

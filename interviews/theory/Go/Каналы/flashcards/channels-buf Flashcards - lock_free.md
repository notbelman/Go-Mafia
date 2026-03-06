#flashcards/channels-buf/lock_free

Как работает lock-free fast path в `select` с `default`? Что именно проверяется и как?
?
![[Lock-free Fast Paths#^lfp-mechanism]]

Почему `select { case v := <-ch: ... default: ... }` не захватывает мьютекс при пустом канале?
?
![[Lock-free Fast Paths#^lfp-mechanism]]

В чём преимущество lock-free fast path для non-blocking операций с каналом?
?
![[Lock-free Fast Paths#^lfp-benefit]]

Что выведет этот код?
```go
ch := make(chan int, 2)
ch <- 1

select {
case v := <-ch:
    fmt.Println("got", v)
default:
    fmt.Println("empty")
}

select {
case ch <- 99:
    fmt.Println("sent")
default:
    fmt.Println("full")
}
```
?
`got 1` затем `sent` — первый select: буфер не пуст, atomic check qcount>0 проходит, берём 1. Второй select: буфер не полон (qcount=0 < dataqsiz=2), atomic check проходит, пишем 99.
![[Lock-free Fast Paths#^lfp-mechanism]]

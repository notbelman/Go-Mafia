#flashcards/channels-buf/send_scenarios

Опиши по шагам что происходит при `ch <- B` когда буфер не полон и никто не ждёт в recvq.
?
![[1 -  буфер не полон, никто не ждёт#^send1-steps]]

Какие три операции выполняются под мьютексом при обычной записи в буфер (сценарий: буфер не полон, recvq пуст)?
?
![[1 -  буфер не полон, никто не ждёт#^send1-mutex-ops]]

Что такое Direct Copy при отправке? Когда он возникает и в чём оптимизация?
?
![[2 - буфер пуст + receiver ждёт (Direct Copy)#^send2-desc]]

Опиши по шагам что происходит при `ch <- D` когда буфер пуст и в recvq есть ждущий receiver (Direct Copy).
?
![[2 - буфер пуст + receiver ждёт (Direct Copy)#^send2-steps]]

При Direct Copy (send): куда именно sender записывает данные? Как это физически работает?
?
![[2 - буфер пуст + receiver ждёт (Direct Copy)#^send2-elem-ptr]]

Почему при Direct Copy буфер не трогается, даже если канал буферизированный?
?
![[2 - буфер пуст + receiver ждёт (Direct Copy)#^send2-optimization]]

Опиши по шагам что происходит при `ch <- D` когда буфер полон и никто не ждёт в recvq.
?
![[3 - буфер полон, никто не ждёт#^send3-steps]]

Что хранит `sudog` sender? Что означает `elem = &D`?
?
![[3 - буфер полон, никто не ждёт#^send3-sudog]]

Как receiver пробуждает спящего sender (сценарий send3)? Что такое Handoff в этом контексте?
?
![[3 - буфер полон, никто не ждёт#^send3-handoff]]

Что выведет этот код?
```go
ch := make(chan int, 1)
ch <- 100

var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    ch <- 200
    fmt.Println("sender done")
}()
time.Sleep(10 * time.Millisecond)
fmt.Println(<-ch)
fmt.Println(<-ch)
wg.Wait()
```
?
`100` `200` `sender done` — первый recv: буфер не пуст, читаем 100 (recv1). При чтении 100: буфер полон и sender ждёт → Handoff, 200 кладётся в буфер, горутина будится. Второй recv: читаем 200.
![[3 - буфер полон, никто не ждёт#^send3-handoff]]

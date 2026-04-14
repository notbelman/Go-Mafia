#flashcards/channels-buf/recv_scenarios

Опиши по шагам что происходит при `x := <-ch` когда буфер не пуст и никто не ждёт в sendq.
?
![[1 - буфер не пуст, никто не ждёт#^recv1-steps]]

Какие три операции выполняются под мьютексом при обычном чтении из буфера (сценарий: буфер не пуст, sendq пуст)?
?
![[1 - буфер не пуст, никто не ждёт#^recv1-mutex-ops]]

Что такое паттерн Handoff при чтении из канала? При каком условии он возникает?
?
![[2 - буфер полон + sender ждёт (Handoff)#^recv2-desc]]

Опиши по шагам что происходит при `x := <-ch` когда буфер полон и в sendq есть ждущий sender (Handoff).
?
![[2 - буфер полон + sender ждёт (Handoff)#^recv2-steps]]

В сценарии Handoff (recv): буфер был полон, receiver прочитал элемент и разбудил sender. Каким стало значение `qcount` после операции?
?
![[2 - буфер полон + sender ждёт (Handoff)#^recv2-qcount-unchanged]]

Что делает `goready(G1)` в контексте разблокировки sender?
?
![[2 - буфер полон + sender ждёт (Handoff)#^recv2-goready]]

Что такое Direct Copy Bypass при чтении? Почему это оптимизация?
?
![[3 - буфер пуст + sender ждёт (Direct Copy,Bypass, sendDirect())#^recv3-optimization]]

Опиши по шагам что происходит при `x := <-ch` когда буфер пуст и в sendq есть ждущий sender (Direct Copy Bypass).
?
![[3 - буфер пуст + sender ждёт (Direct Copy,Bypass, sendDirect())#^recv3-steps]]

Откуда receiver берёт данные при Direct Copy Bypass? Почему буфер не трогается?
?
![[3 - буфер пуст + sender ждёт (Direct Copy,Bypass, sendDirect())#^recv3-sudog]]

Опиши по шагам что происходит при `x := <-ch` когда буфер пуст и никто не ждёт.
?
![[4 - буфер пуст, никто не ждёт#^recv4-steps]]

Что такое `sudog`? Что хранит `elem` в sudog receiver?
?
![[4 - буфер пуст, никто не ждёт#^recv4-sudog]]

Что делает `gopark()` с горутиной?
?
![[4 - буфер пуст, никто не ждёт#^recv4-gopark]]

Как sender доставляет данные спящему receiver? Куда именно копируются данные?
?
![[4 - буфер пуст, никто не ждёт#^recv4-direct-write]]

Что выведет этот код?
```go
ch := make(chan int, 2)
ch <- 1
ch <- 2

done := make(chan struct{})
go func() {
    fmt.Println(<-ch)
    fmt.Println(<-ch)
    fmt.Println(<-ch)
    close(done)
}()
time.Sleep(10 * time.Millisecond)
ch <- 3
<-done
```
?
`1` `2` `3` — первые два значения из буфера (сценарий recv1). Третье: буфер пуст, горутина заблокирована в recvq (recv4), main пишет 3 и напрямую будит горутину.
![[4 - буфер пуст, никто не ждёт#^recv4-direct-write]]

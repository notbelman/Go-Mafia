```
1. Небуферизированный, одна горутина:
   ch := make(chan int)
   ch <- 1   // ждём receiver, его нет → deadlock

2. Буфер переполнен:
   ch := make(chan int, 1)
   ch <- 1
   ch <- 2   // буфер полон, receiver нет → deadlock

3. Взаимное ожидание:
   go func() { <-ch1; ch2 <- 1 }()
   <-ch2; ch1 <- 1   // оба ждут друг друга → deadlock

4. nil канал без default:
   var ch chan int
   <-ch   // deadlock навечно

5. Пустой select:
   select {}   // deadlock навечно
```
^deadlock-cases

**Паттерн #1 — небуферизированный без пары:** send и receive на небуферизированном канале должны встретиться. В одной горутине это невозможно. ^deadlock-unbuffered-single

**Паттерн #2 — полный буфер:** буферизированный канал блокирует только когда буфер заполнен. Если никто не читает — deadlock. ^deadlock-full-buffer

**Паттерн #3 — взаимное ожидание:** классический deadlock — каждая горутина ждёт другую. ^deadlock-mutual-wait

**Паттерн #4 — nil канал:** операции на nil канале блокируются навсегда (не panic!). ^deadlock-nil-channel

**Паттерн #5 — пустой select:** `select {}` блокирует горутину навсегда, что приводит к deadlock если это main. ^deadlock-empty-select

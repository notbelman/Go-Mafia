#flashcards/channels-patterns/fan_in

Что такое Fan-In (merge)? Что гарантируется, а что нет?
?
![[Fan-In#^fanin-idea]]

Почему в Fan-In нельзя написать wg.Wait() перед return result? Что произойдёт?
?
![[Fan-In#^fanin-close-goroutine]]

Что выведет этот код и в каком порядке?
```go
ch1 := make(chan int, 1)
ch2 := make(chan int, 1)
ch1 <- 1
ch2 <- 2
for v := range merge(ch1, ch2) {
    fmt.Println(v)
    // merge закроет канал после ch1 и ch2
}
```
?
Либо `1 2`, либо `2 1` — порядок не гарантирован, зависит от планировщика. Fan-In запускает по одной горутине на канал, обе конкурентно пишут в result.
![[Fan-In#^fanin-order]]

Опиши типовой паттерн "создал-вернул-асинхронно процессишь" и где он используется.
?
![[Fan-In#^fanin-pattern]]

#flashcards/channels-patterns/fanout_tee

В чём принципиальное различие между Fan-Out и Tee?
?
![[Fan-Out и Tee#^fanout-tee-compare]]

Что такое Fan-Out? Какой алгоритм распределения используется в базовой реализации?
?
![[Fan-Out и Tee#^fanout-def]]

Что произойдёт в Fan-Out если один из потребителей тормозит? Как это исправить?
?
![[Fan-Out и Tee#^fanout-blocking]]

Когда использовать Fan-Out? Приведи примеры.
?
![[Fan-Out и Tee#^fanout-when]]

Что такое Tee? Что произойдёт если один потребитель медленный?
?
![[Fan-Out и Tee#^tee-def]]
![[Fan-Out и Tee#^tee-blocking]]

Когда использовать Tee? Приведи примеры.
?
![[Fan-Out и Tee#^tee-when]]

Что выведет этот код?
```go
in := make(chan int, 3)
in <- 1; in <- 2; in <- 3
close(in)
outs := split(in, 2)  // fan-out round-robin
go func() { for v := range outs[0] { fmt.Print("A:", v, " ") } }()
for v := range outs[1] { fmt.Print("B:", v, " ") }
```
?
`A:1 B:2 A:3` (или с небольшими вариациями порядка вывода между горутинами) — round-robin распределяет: индекс 0→outs[0], индекс 1→outs[1], индекс 2→outs[0]. Каждое значение уходит ровно в один канал.
![[Fan-Out и Tee#^fanout-def]]

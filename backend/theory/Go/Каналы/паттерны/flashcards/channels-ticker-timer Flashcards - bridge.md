#flashcards/channels-patterns/bridge

Что такое Bridge? Какую структуру данных разворачивает?
?
![[Bridge#^bridge-problem]]

Как работает Bridge внутри? Опиши два вложенных цикла.
?
![[Bridge#^bridge-impl]]

Что выведет этот код?
```go
chanOfChans := make(chan (<-chan string))
go func() {
    defer close(chanOfChans)
    ch1 := make(chan string, 2); ch1 <- "a"; ch1 <- "b"; close(ch1)
    ch2 := make(chan string, 2); ch2 <- "c"; ch2 <- "d"; close(ch2)
    chanOfChans <- ch1
    chanOfChans <- ch2
}()
for val := range bridge(chanOfChans) { fmt.Print(val, " ") }
```
?
`a b c d` — Bridge вычитывает **последовательно**: сначала весь ch1 (a, b), потом весь ch2 (c, d). Порядок внутри одного канала гарантирован, порядок между каналами определяется порядком отправки в chanOfChans.
![[Bridge#^bridge-sequential]]

Чем Bridge отличается от Fan-In?
?
![[Bridge#^bridge-sequential]]
![[Bridge#^bridge-parallel-opt]]

Что произойдёт если внутренний канал не закрыт в Bridge?
?
![[Bridge#^bridge-close-required]]

Как можно улучшить базовый Bridge для повышения производительности?
?
![[Bridge#^bridge-parallel-opt]]

Когда применяется Bridge? Приведи примеры.
?
![[Bridge#^bridge-when]]

#flashcards/channels-patterns/pipeline

Что такое Pipeline в Go и из чего состоит каждая стадия?
?
![[Pipeline#^pipeline-idea]]

Что выведет этот код?
```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums { out <- n }
    }()
    return out
}
func double(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for v := range in { out <- v * 2 }
    }()
    return out
}
func main() {
    for v := range double(double(gen(1, 2, 3))) {
        fmt.Println(v)
    }
}
```
?
`4 8 12` — каждое число проходит через два `double`: 1→2→4, 2→4→8, 3→6→12. Стадии выполняются конкурентно, данные текут через каналы.
![[Pipeline#^pipeline-declarative]]

Почему pipeline позволяет декларативный стиль написания кода?
?
![[Pipeline#^pipeline-declarative]]

Как параллелизовать тормозящую стадию pipeline? Опиши механизм.
?
![[Pipeline#^pipeline-parallelize]]

Почему безопасно, что несколько горутин читают из одного входного канала в parallelSend?
?
![[Pipeline#^pipeline-fanin-wg]]

Как WaitGroup используется для закрытия выходного канала при параллелизации стадии?
?
![[Pipeline#^pipeline-fanin-wg]]

Приведи реальный пример применения pipeline с параллелизацией тормозящей стадии.
?
![[Pipeline#^pipeline-real-example]]

Назови плюсы pipeline-архитектуры.
?
![[Pipeline#^pipeline-pros]]

Назови минусы pipeline-архитектуры.
?
![[Pipeline#^pipeline-cons]]

Когда стоит использовать pipeline, а когда нет?
?
![[Pipeline#^pipeline-when]]

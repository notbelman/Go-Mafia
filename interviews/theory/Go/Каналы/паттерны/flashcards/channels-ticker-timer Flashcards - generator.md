#flashcards/channels-patterns/generator

Что такое Generator? В чём его ленивость?
?
![[Generator#^gen-channel-lazy]]

Два способа реализовать генератор в Go. В чём разница?
?
![[Generator#^gen-compare]]

Как работает замыкание-генератор? Почему n живёт между вызовами?
?
![[Generator#^gen-closure]]

Когда выбрать замыкание, а когда канал для генератора?
?
![[Generator#^gen-when]]

Что выведет этот код?
```go
func counter(start int) func() int {
    n := start
    return func() int {
        r := n; n++; return r
    }
}
g1 := counter(0)
g2 := counter(0)
fmt.Println(g1(), g1(), g2())
```
?
`0 1 0` — каждый вызов counter создаёт независимое замыкание со своей переменной `n`. g1 и g2 не делят состояние. g1() дважды → 0, 1; g2() → 0.
![[Generator#^gen-closure]]

Какая проблема с бесконечным канальным генератором? Как её решить?
?
![[Generator#^gen-infinite-leak]]

Что выведет этот код?
```go
fib := fibonacci() // бесконечный генератор
for i := 0; i < 5; i++ {
    fmt.Println(<-fib)
}
// программа завершается
```
?
`0 1 1 2 3` — первые 5 чисел Фибоначчи. После выхода из цикла горутина генератора **утечёт** — она заблокирована на `ch <- a`, читателя нет, GC не соберёт пока горутина жива.
![[Generator#^gen-infinite-leak]]

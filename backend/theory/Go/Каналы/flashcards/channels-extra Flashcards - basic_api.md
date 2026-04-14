#flashcards/channels-extra/basic_api

Назови 4 базовые операции с каналом в Go. Есть ли что-то пятое?
?
![[Базовый API и range#^api-four-ops]]

Что означает `ok=false` при чтении `v, ok := <-ch`? Каковы оба условия?
?
![[Базовый API и range#^api-ok-semantics]]

Буферизированный канал: `make(chan int, 2)`, записали 10 и 20, закрыли. Что вернут три последовательных чтения `v, ok := <-ch`?
?
![[Базовый API и range#^api-ok-buffered]]

Почему `range` по незакрытому каналу — утечка горутины?
?
![[Базовый API и range#^api-range-no-close-leak]]

Что выведет этот код?
```go
ch := make(chan int, 2)
ch <- 10
ch <- 20
close(ch)
for v := range ch {
    fmt.Println(v)
}
fmt.Println("done")
```
?
`10`, `20`, `done` — range дочитывает буфер до опустошения, потом завершается (ok=false под капотом). Канал закрыт, поэтому range не блокируется.
![[Базовый API и range#^api-ok-buffered]]

Что выведет этот код?
```go
ch := make(chan int, 1)
ch <- 42
close(ch)
v1, ok1 := <-ch
v2, ok2 := <-ch
fmt.Println(v1, ok1)
fmt.Println(v2, ok2)
```
?
`42 true` и `0 false` — первое чтение достаёт значение из буфера (ok=true), второе читает из пустого закрытого канала (zero value, ok=false).
![[Базовый API и range#^api-ok-buffered]]

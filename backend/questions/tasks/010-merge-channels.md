---
type: task
companies:
  - OZON
topic: Go
subtopic:
  - Channels 
  - Concurrency
  -  WaitGroup
title: Merge N каналов в один
---

## Условие
Написать код функции, которая делает merge N каналов. Весь входной поток перенаправляется в один канал.

```go
func merge(cs ...<-chan int) <-chan int {
    ...
}
```

## Решение

```go
func merge(cs ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    for _, ch := range cs {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for val := range c {
                out <- val
            }
        }(ch)
    }
    
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}
// - Зачем канал передаётся в горутину явно как параметр?
// - Зачем закрывать исходящий канал в отдельной горутине через WaitGroup?
// - Что изменится, если убрать закрытие канала?
// - Почему канал закрывает эта функция, а не вызывающий?
```
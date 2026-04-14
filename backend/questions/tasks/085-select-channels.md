---
type: task
companies:
  - Uzum
topic: Go
subtopic:
  - Channels
  - Select
  - Buffered Channel
title: Select с записью и чтением из одного буферизированного канала
---

## Условие

Что выведет данный код в консоль и почему? Что изменится, если поменять кейсы местами или увеличить буфер?

## Пример
```go
package main

func main() {
    ch := make(chan int, 1)
    for i := 0; i < 10; i++ {
        select {
        case x := <-ch:
            println(x)
        case ch <- i:
        }
    }
}
```

## Решение
```go
```
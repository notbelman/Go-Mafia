---
type: task
companies:
  - Магнит
topic: Go
subtopic:
  - Select
  - Buffered Channels
title: Select с двумя буферизированными каналами — что выведет Result?
---

## Условие

Что выведет программа? Какие варианты результата возможны и почему?

## Пример
```go
package main

import "fmt"

func main() {
    var (
        ch1 = make(chan int, 1)
        ch2 = make(chan int, 1)
        res = make([]int, 0, 2)
    )

    ch1 <- 1
    ch2 <- 2

    select {
    case msg := <-ch2:
        res = append(res, msg)
    case msg := <-ch1:
        res = append(res, msg)
    }

    fmt.Printf("Result: %v\n", res)
}
```

## Решение
```go
```
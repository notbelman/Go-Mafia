---
type: task
companies:
  - GroupIB
topic: Go
subtopic:
  - Channels
  - Select
  - Goroutines
title: Что выведет программа с select и двумя каналами
---

## Условие

Что выводит программа в стандартный поток вывода?

## Пример
```go
package main

import "fmt"

func callCallbacks(a, b func()) {
    go a()
    go b()
}

func main() {
    example()
}

func example() {
    firstDone := make(chan struct{})
    secondDone := make(chan struct{})

    callCallbacks(
        func() {
            fmt.Printf("a")
            close(firstDone)
        },
        func() {
            fmt.Printf("b")
            close(secondDone)
        },
    )

    count := 0
    for count < 2 {
        select {
        case <-firstDone:
            count++
        case <-secondDone:
            count++
        }
    }

    fmt.Printf("%d", count)
}
```

## Решение
```go
```
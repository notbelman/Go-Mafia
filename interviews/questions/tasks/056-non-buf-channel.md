---
type: task
companies:
  - МВидео
topic: Go
subtopic:
  - Channels
  - Goroutines
  - Close
title: Запись в небуферизированный канал, закрытие и range
---

## Условие

Что выведет программа? В какой момент и на какой строке программа сломается?

## Пример
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int)
    go func() {
        ch <- 1
    }()
    time.Sleep(time.Millisecond * 500)
    close(ch)

    for i := range ch {
        fmt.Println(i)
    }

    time.Sleep(time.Millisecond * 100)
}
```

## Решение
```go
```
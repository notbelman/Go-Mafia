---
type: task
companies:
  - Uzum
topic: Go
subtopic:
  - Panic
  - Recover
  - Goroutines
  - Defer
title: Recover в main не ловит панику из другой горутины
---

## Условие

Что выведет данный код в консоль и почему? Долго ли будет выполняться?

## Пример
```go
package main

import "time"

func main() {
    defer func() {
        if err := recover(); err != nil {
            println(err)
        }
    }()

    go func() {
        panic(123)
    }()

    time.Sleep(time.Hour)
}
```

## Решение
```go
```
---
type: task
companies:
  - Flant
topic: Go
subtopic:
  - WaitGroup
  - Channels
  - Select
  - Deadlock
title: WaitGroup + select без default — найди deadlock и исправь
---

## Условие

Есть ли ошибка в коде? Почему из цикла for никогда не выйдет управление? Как называется такая ситуация и как исправить?

## Пример
```go
package main

import (
    "fmt"
    "strconv"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    c := make(chan string, 3)

    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(c chan<- string, i int, group *sync.WaitGroup) {
            defer wg.Done()
            c <- fmt.Sprintf("Goroutine %s", strconv.Itoa(i))
        }(c, i, &wg)
    }

    for {
        select {
        case v := <-c:
            fmt.Println(v)
        }
    }

    wg.Wait()
    close(c)
}
```

## Решение
```go
```
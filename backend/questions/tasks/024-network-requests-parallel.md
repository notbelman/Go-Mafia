---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Concurrency
  -  WaitGroup
  - Atomic
title: Network requests - параллельные сетевые запросы
---

## Условие
Что выведет следующая программа и сколько она будет выполняться по времени?

```go
package main

import (
    "fmt"
    "time"
)

const numRequests = 10000

var count int

func networkRequest() {
    time.Sleep(time.Millisecond) // Эмуляция сетевого запроса (~1 мс)
    count++
}

func main() {
    for i := 0; i < numRequests; i++ {
        networkRequest()
    }
    fmt.Println(count)
}
```

**Ответ:** Программа выведет `10000`. Время выполнения ~10 секунд (10000 × 1мс последовательно).

## Решение

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"
)

const numRequests = 10000

var count int32
var wg sync.WaitGroup

func networkRequest() {
    defer wg.Done()
    time.Sleep(time.Millisecond)
    atomic.AddInt32(&count, 1)
}

func main() {
    wg.Add(numRequests)
    for i := 0; i < numRequests; i++ {
        go networkRequest()
    }
    wg.Wait()
    fmt.Println(count)
}
```

**Время выполнения:** ~10-50 мс (все горутины параллельно, каждая спит 1 мс).

## Дополнительные вопросы

**Что будет, если вызывать `wg.Add(1)` внутри горутины?**

```go
func main() {
    for i := 0; i < numRequests; i++ {
        go func() {
            wg.Add(1)
            networkRequest()
        }()
    }
    wg.Wait()
    fmt.Println(count)
}
```

**Ответ:** Так делать нельзя — `main` может дойти до `wg.Wait()` до того, как `Add(1)` успеет выполниться. `Wait()` увидит счётчик равным нулю и сразу завершится.

**Что будет, если вызывать `wg.Add(-1)` в цикле перед запуском горутин?**

```go
func main() {
    for i := 0; i < numRequests; i++ {
        wg.Add(-1)
        go func() {
            networkRequest()
        }()
    }
    wg.Wait()
    fmt.Println(count)
}
```

**Ответ:** Будет паника — `sync.WaitGroup` не допускает отрицательных значений счётчика. `Add(-1)` уменьшает счётчик, а он ещё равен нулю.

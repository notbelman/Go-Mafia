---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Concurrency
  - Race Condition
  - Atomic
  - Mutex
title: Найти максимальное четное число - race condition
explained: "true"
---

## Условие
Что не так с этим кодом? Как исправить?

```go
package main

import "fmt"

func main() {
    var max int
    for i := 1000; i > 0; i-- {
        go func() {
            if i%2 == 0 && i > max {
                max = i
            }
        }()
    }
    fmt.Printf("Maximum is %d", max)
}
```

**Проблема 1: Захват переменной `i` по ссылке (до Go 1.22)** Все 1000 горутин захватывают одну и ту же переменную `i`, а не её значение в момент создания. Горутины запускаются асинхронно — к моменту когда горутина реально начнёт выполняться, цикл уже мог уйти дальше. Большинство горутин увидят `i = 0` (конечное значение цикла). С Go 1.22 переменная цикла создаётся заново на каждой итерации — проблема исчезает.

**Проблема 2: Race condition при чтении/записи `max`** Множество горутин одновременно читают и пишут в `max` без синхронизации. Одна горутина читает `max` для сравнения, в это время другая пишет в `max` новое значение. Результат непредсказуем, `go run -race` покажет ошибку.

**Проблема 3: Нет ожидания завершения горутин** `fmt.Printf` выполняется сразу после запуска горутин, не дожидаясь их завершения. `main` завершается → программа умирает → горутины не успевают отработать. `max` скорее всего будет 0.

## Решение

**Способ 1: Mutex**

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var max int
    var mu sync.Mutex
    var wg sync.WaitGroup

    for i := 1000; i > 0; i-- {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            if n%2 == 0 {
                mu.Lock()
                if n > max {
                    max = n
                }
                mu.Unlock()
            }
        }(i)
    }

    wg.Wait()
    fmt.Printf("Maximum is %d\n", max)
}
```

**Способ 2: Atomic (CAS)**

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var max int64
    var wg sync.WaitGroup

    for i := 1000; i > 0; i-- {
        wg.Add(1)
        go func(n int64) {
            defer wg.Done()
            if n%2 == 0 {
                for {
                    old := atomic.LoadInt64(&max)
                    if n <= old {
                        return
                    }
                    if atomic.CompareAndSwapInt64(&max, old, n) {
                        return
                    }
                }
            }
        }(int64(i))
    }

    wg.Wait()
    fmt.Printf("Maximum is %d\n", max)
}
```

**Способ 3: Atomic (Load/Store)**

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var max int64
    var wg sync.WaitGroup

    for i := 1000; i > 0; i-- {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            if i%2 == 0 {
                currentMax := atomic.LoadInt64(&max)
                if int64(i) > currentMax {
                    atomic.StoreInt64(&max, int64(i))
                }
            }
        }(i)
    }

    wg.Wait()
    fmt.Printf("Maximum is %d", max)
}
```

**Примечание:** Способ 3 имеет race condition между Load и Store — CAS надёжнее.

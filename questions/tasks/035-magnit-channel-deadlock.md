---
type: task
companies:
  - Магнит
topic: Go
subtopic:
  - Goroutines
  - Channels 
title: Что выведет - канал и rand.Int
---

## Условие
Что выведет программа?

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Printf("Start\n")

    res := make(chan int)
    res <- rand.Int()

    fmt.Printf("Im finish with: %d", <-res)
}
```

## Решение

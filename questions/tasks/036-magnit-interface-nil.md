---
type: task
companies:
  - Магнит
topic: Go
subtopic:
  - Interfaces
  - nil
title: Что выведет - интерфейс и nil
---

## Условие
Что выведет программа?

```go
package main

import "fmt"

type Figure interface {
    Area() float64
}

type square struct {
    a float64
}

func (s *square) Area() float64 {
    return s.a * s.a
}

func main() {
    var a *square

    figure := Figure(a)

    fmt.Printf("Is figure: %t\n", figure == nil)
}
```

## Решение

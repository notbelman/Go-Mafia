---
type: task
companies:
  - Магнит
topic: Go
subtopic:
  - Slice
  - Append
  - Underlying Array
title: Анализ поведения слайса при change и append
---

## Условие

Что выведет программа? Объясни поведение слайсов a, b и c.

## Пример
```go
package main

import "fmt"

func main() {
    a := []int{1, 2, 3} // len = 2, cap = 3
    b := change(a)
    c := append(a, 5)

    fmt.Printf("a: %v\n", a)
    fmt.Printf("b: %v\n", b)
    fmt.Printf("c: %v\n", c)
}

func change(a []int) []int {
    return append(a, 4)
}
```

## Решение
```go
```
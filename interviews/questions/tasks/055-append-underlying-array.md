---
type: task
companies:
  - МВидео
topic: Go
subtopic:
  - Slices
  - Append
title: Поведение append при передаче подслайса с общим underlying array
---

## Условие

Что выведет программа? Объяснить, как append влияет на исходный слайс при передаче подслайса, у которого остался capacity от оригинала.

## Пример
```go
package main

import "fmt"

func subis(is []int) []int {
    return append(is, 5)
}

func main() {
    is := []int{1, 2, 3, 4} // len = 4 cap = 4
    subis(is[2:3])
    fmt.Println(is)
}
```

## Решение
```go
```
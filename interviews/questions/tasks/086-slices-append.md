---
type: task
companies:
  - Uzum
topic: Go
subtopic:
  - Slice
  - Subslice
  - Append
  - Underlying Array
title: Анализ append к подслайсу — влияние на оригинал
---

## Условие

Что выведет данный код в консоль и почему?

## Пример
```go
package main

import "fmt"

func add(s []string) {
    s = append(s, "x")
}

func main() {
    s := []string{"a", "b", "c"}
    add(s[1:2])
    fmt.Println(s)
}
```

## Решение
```go
```
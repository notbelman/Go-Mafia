---
type: task
companies:
  - Uzum
topic: Go
subtopic:
  - Defer
  - Closures
  - LIFO
title: Анализ defer в цикле range — порядок вывода
---

## Условие

Что выведет данный код в консоль и почему?

## Пример
```go
package main

func main() {
    digits := []int{1, 2, 3, 4, 5}
    for _, d := range digits {
        defer println(d)
    }
}
```

## Решение
```go
```
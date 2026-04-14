---
type: task
companies:
  - GroupIB
topic: Go
subtopic:
  - Goroutines
  - Closures
  - Range
title: Что выведется при запуске горутин в цикле range
---

## Условие

Что выведется и в какой последовательности?

## Пример
```go
func main() {
    a := []int{
        1, 2, 3, 4, 5,
    }
    for _, i := range a {
        go func() {
            fmt.Print(i)
        }()
    }
}
```

## Решение
```go
```
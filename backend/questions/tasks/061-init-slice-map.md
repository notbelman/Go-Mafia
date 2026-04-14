---
type: task
companies:
  - GroupIB
topic: Go
subtopic:
  - Slices
  - Map
  - Make
title: Что выведет программа — инициализация слайса и мапы
---

## Условие

Что будет выводом двух программ? Объяснить разницу между объявлением через литерал и через make.

## Пример
```go
// Task 1
func main() {
    a := []int{}
    b := make([]int, 10)
    fmt.Println(a[0])
    fmt.Println(b[0])
}

// Task 2
func main() {
    a := map[string]int{}
    b := make(map[string]int, 10)
    a["test"]++
    b["test"]++
    fmt.Println(a["test"])
    fmt.Println(b["test"])
}
```

## Решение
```go
```
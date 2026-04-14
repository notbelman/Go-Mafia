---
type: task
companies:
  - Амтех
topic: Go
subtopic:
  - Slice
  - Append
  - Subslice
  - Capacity
title: Четыре теста на поведение слайсов — appendToSlice и подслайсы
---

## Условие

Что выведет каждый тест? Будет ли паника в TestSlice3?

## Пример
```go
func TestSlice1(t *testing.T) {
    sl := []int{1, 2, 3}
    appendToSlice(sl)
    fmt.Println(sl)
}

func TestSlice2(t *testing.T) {
    sl := []int{1, 2, 3}
    appendToSlice(sl[0:2])
    fmt.Println(sl)
}

func TestSlice3(t *testing.T) {
    sl := make([]int, 0, 3)
    appendToSlice(sl[0:2])
    // Будет ли паника?
    //sl[0] = 1
    //fmt.Println(sl)
}

func TestSlice4(t *testing.T) {
    sl := make([]int, 0, 3)
    sl = append(sl, []int{1, 2}...)
    appendToSlice(sl[0:2])
    fmt.Println(sl)
}
```

## Решение
```go
```
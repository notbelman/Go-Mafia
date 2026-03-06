---
type: task
companies:
  - Витех
topic: Go
subtopic:
  - Slice
  - Append
  - Capacity
  - Underlying Array
title: Анализ слайсов a и b с общим underlying array и append
---

## Условие

Что выведет программа и почему?

## Пример
```go
a := make([]int, 0, 2)
b := a
a = append(a, 1)
b = append(b, 2)
println(a[0], b[0])
```

## Решение
```go
```
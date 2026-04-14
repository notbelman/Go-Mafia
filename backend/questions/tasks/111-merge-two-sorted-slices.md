---
type: task
companies:
  - Астрал-Софт
topic: Algorithms
subtopic:
  - Merge
  - Two Pointers
  - Sorted Arrays
title: Слияние двух отсортированных слайсов в один отсортированный
---

## Условие

На входе 2 отсортированных по возрастанию слайса. Реализовать функцию, возвращающую на выходе отсортированный по возрастанию слайс, состоящий из элементов a и b.

## Пример
```
a = [1, 3, 5], b = [2, 4, 6] -> [1, 2, 3, 4, 5, 6]
```

## Решение

```go
func merge(a, b []int) []int {
    result := make([]int, 0, len(a)+len(b))
    i, j := 0, 0

    for i < len(a) && j < len(b) {
        if a[i] <= b[j] {
            result = append(result, a[i])
            i++
        } else {
            result = append(result, b[j])
            j++
        }
    }

    result = append(result, a[i:]...)
    result = append(result, b[j:]...)

    return result
}
```

**Сложность:** O(n+m) по времени, O(n+m) по памяти.

Классический two pointers на двух отсортированных массивах — основа merge sort. Цикл идёт пока оба указателя в пределах массивов, после цикла дописываем остатки через `a[i:]...` и `b[j:]...` (один из них будет пустым).

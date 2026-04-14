---
type: task
companies:
  - Yandex
topic: Algorithms
subtopic:
  - Sliding Window
  - Two Pointers
  - Arrays
title: Максимальный подынтервал единиц при удалении одного элемента
---

## Условие

Дан непустой слайс из нулей и единиц. Нужно определить, какой максимальный по длине подынтервал единиц можно получить, удалив (пропустив) ровно один элемент массива.

Удалять один элемент из слайса обязательно.

## Пример
```
longestSubarray([]int{1, 1, 0, 1}) == 3
longestSubarray([]int{1, 1, 0, 0, 1}) == 2
longestSubarray([]int{1, 0, 1, 0, 0, 1}) == 2
longestSubarray([]int{1, 0, 0, 1, 0, 0, 1}) == 1
longestSubarray([]int{0, 1, 1, 0, 1, 1, 0, 1, 0, 0, 1}) == 4
longestSubarray([]int{0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 0, 1}) == 5
longestSubarray([]int{1, 1, 1}) == 2
```

## Решение
```go
```
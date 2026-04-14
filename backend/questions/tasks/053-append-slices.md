---
type: task
companies:
  - OZON
topic: Go
subtopic:
  - Slices
  - Append
title: Поведение append при передаче подслайса в функцию
---

## Условие

Что выведет данный код? Объяснить поведение append при передаче подслайса в функцию — когда перезаписывается исходный массив, а когда нет.

## Пример
```go
func main() {
    nums := []int{1, 2, 3}

    addNum(nums[0:2])
    fmt.Println(nums) // ?

    addNums(nums[0:2])
    fmt.Println(nums) // ?
}

func addNum(nums []int) {
    nums = append(nums, 4)
}

func addNums(nums []int) {
    nums = append(nums, 5, 6)
}
```

## Решение
```go
```
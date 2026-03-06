---
type: task
companies:
  - X5
topic: Go
subtopic:
  - Slices
  - Copy
  - Append
title: Что выведет - операции со слайсами
---

## Условие
Что выведет программа?

```go
import (
    "fmt"
)

func main() {
    nums := make([]int, 1, 3)
    fmt.Println(nums) // что выведет?

    appendSlice(nums, 1)
    fmt.Println(nums) // что выведет?

    copySlice(nums, []int{2, 3})
    fmt.Println(nums) // что выведет?

    mutateSlice(nums, 1, 4)
    fmt.Println(nums) // что выведет?
}

func appendSlice(sl []int, val int) {
    sl = append(sl, val)
}

func copySlice(sl, cp []int) {
    copy(sl, cp)
}

func mutateSlice(sl []int, idx, val int) {
    sl[idx] = val
}
```

## Решение

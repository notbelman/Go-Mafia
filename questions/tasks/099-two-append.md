---
type: task
companies:
  - Flant
topic: Go
subtopic:
  - Slice
  - Append
  - Capacity
  - Underlying Array
title: Два append от одного слайса — y и z после append(x, 3) и append(x, 4)
explained: "true"
---

## Условие

Что выведет программа и почему?

## Пример
```go
package main

import "fmt"

func main() {
    x := []int{}
    x = append(x, 0)
    x = append(x, 1)
    x = append(x, 2)
    y := append(x, 3)
    z := append(x, 4)
    fmt.Println(y, z) // ?
}
```

## Решение
```go
    x := []int{}           // x = [] len=0 cap=0
    x = append(x, 0)       // x = [0] len = 1, cap = 1
    x = append(x, 1)       // x = [0, 1] len=2 cap=2
    x = append(x, 2)       // x = [0, 1, 2] len=3 cap=4 (capacity выросла)
    y := append(x, 3)      // y = [0, 1, 2, 3] len=4 cap=4, добавляется 3 в тот же массив
    z := append(x, 4)      // z = [0, 1, 2, 4] len=4 cap=4, 3 заменяется на 4 (тот же массив)
    fmt.Println(y, z)
```

**Ответ:** `[0 1 2 4] [0 1 2 4]`

3 заменилось на 4, потому что append добавляет новый элемент в конец слайса после текущей длины. Длина слайса x была 3, и при добавлении 4, слайс z скопировал длину 3 и добавил 4 в новый индекс, который идет после 3-го элемента.

Оба слайса `y` и `z` ссылаются на один и тот же underlying array, поэтому запись в `z` перезаписала значение, которое было записано через `y`.
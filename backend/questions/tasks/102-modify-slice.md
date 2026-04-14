---
type: task
companies:
  - Wildberries
topic: Go
subtopic:
  - Slice
  - Append
  - Range
  - Underlying Array
title: Modify слайса в range с append при чётных индексах
explained: "true"
---

## Условие

Что выведет код и почему?

## Пример
```go
func main() {
    s := []int{1, 2, 3}
    modify(s)
    fmt.Println(s)
}

func modify(s []int) {
    for i, n := range s {
        s[i] = n * 2
        if i%2 == 0 {
            s = append(s, i*2)
        }
    }
}
```

## Решение
```go
func main() {
    s := []int{1, 2, 3} // s.len = 3, s.cap = 3
    modify(s)
    fmt.Println(s)
}

func modify(s []int) {
    for i, n := range s { // итерация index, num
        s[i] = n * 2 
        if i%2 == 0 { // при четном количестве у нас будет выполняться append
            s = append(s, i*2) // при первом же append будет реалокация
		    // len + 1 > cap -> 3 + 1 > 3
        }
    }
}
```

### Почему будет реаллокация и как работает append
![[append#Как append работает внутри]]
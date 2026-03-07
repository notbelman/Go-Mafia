---
type: task
companies:
  - Wildberries
topic: Go
subtopic:
  - Map
  - Race Condition
  - Goroutines
  - Mutex
title: Конкурентная запись в мапу из 100 горутин — data race
---

## Условие

Что произойдёт при запуске? Как исправить?

## Пример
```go
func main() {
    data := map[string]int{
        "value": 0,
    }

    grow := func(a map[string]int) {
        a["value"]++
    }

    for i := 0; i < 100; i++ {
        go grow(data)
    }
    fmt.Println(data["value"])
}
```

## Решение
```go
```
---
type: task
companies:
  - Витех
topic: Go
subtopic:
  - Goroutines
  - Race Condition
  - Atomic
  - Mutex
title: Инкремент счётчика в 1000 горутинах — race condition
---

## Условие

Что будет на выходе? Как исправить? Можно ли сохранить последовательность без мьютексов?

## Пример
```go
var counter int
for i := 0; i < 1000; i++ {
    go func() {
        counter++
    }()
}
println(counter)
```

## Решение
```go
```
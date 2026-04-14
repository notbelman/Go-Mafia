---
type: task
companies:
  - OZON
topic: Go
subtopic:
  - Channels
  - Goroutines
  - Concurrency
  - Time
title: Два последовательных чтения из worker() — сколько секунд выполняется
---

## Условие

Что выведет программа и почему? Сколько секунд займёт выполнение?

## Пример
```go
func main() {
    timeStart := time.Now()
    _, _ = <-worker(), <-worker()
    println(int(time.Since(timeStart).Seconds()))
}

func worker() chan int {
    ch := make(chan int)
    go func() {
        time.Sleep(3 * time.Second)
        ch <- 1
    }()
    return ch
}
```

## Решение
```go
```
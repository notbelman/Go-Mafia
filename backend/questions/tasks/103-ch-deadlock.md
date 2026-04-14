---
type: task
companies:
  - Wildberries
topic: Go
subtopic:
  - Channels
  - WaitGroup
  - Deadlock
title: Deadlock с небуферизированным каналом и двумя deposit-горутинами
---

## Условие

Что выведет код? Есть ли дедлок? Как починить?

## Пример
```go
var (
    balance int
)

func init() {
    balance = 100
}

func deposit(val int, wg *sync.WaitGroup, ch chan bool) {
    ch <- true
    balance += val
    <-ch
    wg.Done()
}

func main() {
    var wg sync.WaitGroup
    ch := make(chan bool)
    wg.Add(2)
    go deposit(10, &wg, ch)
    go deposit(20, &wg, ch)
    wg.Wait()
    fmt.Println("Balance is: ", balance)
}
```

## Решение
```go
```
---
type: task
companies:
  - Сбер
topic: Go
subtopic:
  - Channels
  - Goroutines
  - WaitGroup
title: Слить N каналов в один (fan-in)
---

## Условие

Даны N каналов типа `chan int`. Надо написать функцию, которая смерджит все данные из этих каналов в один и вернёт его.

Также нужно написать заполнение каналов значениями.

## Пример
```
### in
a := make(chan int)
b := make(chan int)
c := make(chan int)
// заполнить a, b, c значениями

### out
for num := range joinChannels(a, b, c) {
    fmt.Println(num)
}
// все значения из a, b, c в произвольном порядке
```

## Решение
```go
```
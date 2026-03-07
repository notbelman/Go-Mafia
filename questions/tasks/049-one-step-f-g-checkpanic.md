---
type: task
companies:
  - Сбер
topic: Go
subtopic:
  - Defer
  - Panic
  - Recover
  - Iterators
title: Анализ кода с defer, panic, recover и итераторами
---

## Условие

Что выведет данный код? Объяснить порядок выполнения defer, panic и recover, а также поведение итератора oneStep и цикла for range.

## Пример
```go
package main

func oneStep(yield func() bool) {
    yield()
}

func f() {
    for range oneStep {
        print("f ")
        recover()
    }
}

func g() {
    for range 1 {
        print("g ")
        recover()
    }
}

func checkPanic(loop func(), c chan<- struct{}) {
    defer func() {
        println(recover() != nil)
        c <- struct{}{}
    }()
    defer loop()
    panic(123)
}

func main() {
    c := make(chan struct{}, 1)
    go checkPanic(f, c)
    <-c
    go checkPanic(g, c)
    <-c
}
```

## Решение
```go
```
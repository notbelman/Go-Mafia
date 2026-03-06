---
type: task
companies:
  - Магнит
topic: Go
subtopic:
  - Concurrency
  - Channels 
  - Race Condition
title: Почему sumIntoChannels быстрее sumIntoVars
---

## Условие
Представим, что у нас есть машина с двумя ядрами. Нам надо сложить значения из слайса Input'ов и вернуть сумму по соответствующим полям. Мы написали две функции, потом бенчмарки к ним.

**Вопрос:** Почему `sumIntoChannels` сильно быстрее, чем `sumIntoVars`?

```go
package main

import (
    "sync"
)

type Input struct {
    a int64
    b int64
}

type Result struct {
    sumA int64
    sumB int64
}

func sumIntoVars(inputs []Input) Result {
    var (
        sumA int64
        sumB int64
        wg   sync.WaitGroup
    )

    wg.Add(2)

    go func() {
        defer wg.Done()

        for _, input := range inputs {
            sumA += input.a
        }
    }()

    go func() {
        defer wg.Done()

        for _, input := range inputs {
            sumB += input.b
        }
    }()

    wg.Wait()

    return Result{
        sumA: sumA,
        sumB: sumB,
    }
}

func sumIntoChannels(inputs []Input) Result {
    chA := make(chan int64)
    chB := make(chan int64)

    go func() {
        var sumA int64

        for _, input := range inputs {
            sumA += input.a
        }

        chA <- sumA
    }()

    go func() {
        var sumB int64

        for _, input := range inputs {
            sumB += input.b
        }

        chB <- sumB
    }()

    sumA := <-chA
    sumB := <-chB

    return Result{
        sumA: sumA,
        sumB: sumB,
    }
}
```

## Решение

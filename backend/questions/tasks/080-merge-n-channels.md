---
type: task
companies:
  - Uzum
topic: Go
subtopic:
  - Channels
  - WaitGroup
  - Fan-In
  - Generics
title: Fan-in — слияние N каналов в один (пакет chan_utils)
---

## Условие

Написать функцию в пакете chan_utils, которая принимает произвольное количество каналов и возвращает один канал того же типа со всеми значениями из входных каналов.

Дополнительно: есть ли ограничения по версии Go? Можно ли сделать обобщённо через дженерики?

## Пример
```go
package chan_utils

// func: input N-chans; output: 1; all values in output;
```

## Решение
```go
```
---
type: task
companies:
  - Uzum
topic: Go
subtopic:
  - sync.Once
  - Mutex
  - Atomic
  - Thread Safety
title: Кастомная реализация sync.Once с гарантиями потокобезопасности
---

## Условие

Реализовать структуру Once, которая выполняет переданную функцию только один раз и обеспечивает потокобезопасность.

Гарантии:
1. Функция выполняется ровно 1 раз
2. Потокобезопасность

## Пример
```go
package sync // custom stdlib

type Once struct {
    mu  m
    abc mu
}

// guarantee:
// 1] fn x 1 time
// 2] threadsafe
func (o *Once) Do(fn func()) {

}
```

## Решение
```go
```
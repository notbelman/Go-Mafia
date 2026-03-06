---
type: task
companies:
  - МВидео
topic: Go
subtopic:
  - Errors
  - Interface
  - Nil
title: Кастомная ошибка и сравнение с nil через интерфейс
---

## Условие

Что выведет программа? Объяснить поведение сравнения кастомной структуры ошибки с nil при возврате через интерфейс error.

## Пример
```go
package main

import "fmt"

type MyError struct {
    data string
}

func (e MyError) Error() string {
    return e.data
}

func main() {
    err := foo(4)
    if err != nil {
        fmt.Println("oops")
    } else {
        fmt.Println("ok")
    }
}

func foo(i int) error {
    var err *MyError
    if i > 5 {
        err = &MyError{data: "i > 5"}
    }
    return err
}
```

## Решение
```go
```
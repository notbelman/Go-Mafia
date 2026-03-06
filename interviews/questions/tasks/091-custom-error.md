---
type: task
companies:
  - Амтех
topic: Go
subtopic:
  - Interface
  - Nil
  - Error
title: CustomError и nil — почему err != nil при возврате nil *CustomError
---

## Условие

Что выведет программа? Чему будет равен err и почему?

## Пример
```go
import (
    "fmt"
    "testing"
)

type CustomError struct {
    Message string
}

func (e *CustomError) Error() string {
    return e.Message
}

func getError() *CustomError {
    return nil
}

func TestError(t *testing.T) {
    var err error
    err = getError()
    if err != nil {
        fmt.Println("Error is not nil")
    }
}
```

## Решение
```go
```
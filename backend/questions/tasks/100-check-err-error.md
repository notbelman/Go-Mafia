---
type: task
companies:
  - Flant
topic: Go
subtopic:
  - Interface
  - Nil
  - Error
title: checkErr и nil — четыре варианта присваивания error-интерфейсу
---

## Условие

Что выведет каждый вызов checkErr и почему?

## Пример
```go
package main

import "fmt"

type errorString struct {
    s string
}

func (e errorString) Error() string {
    return e.s
}

func checkErr(err error) {
    fmt.Println(err == nil)
}

func main() {
    var e1 error
    checkErr(e1) // ?

    var e *errorString
    checkErr(e) // ?

    e = &errorString{}
    checkErr(e) // ?

    e = nil
    checkErr(e) // ?
}
```

## Решение
```go
```
---
type: task
companies:
  - OZON
topic: Go
subtopic:
  - Interfaces
  - nil
  - Error
title: Ошибка и вывод - nil interface vs nil pointer
explained: "true"
---

## Условие
```go
import "fmt"

func main() {
    err1 := foo(true)
    if err1 != nil {
        fmt.Println("fire error 1")
    }
    err2 := foo(false)
    if err2 != nil {
        fmt.Println("fire error 2")
    }
    fmt.Println("complete")
}

type CustomError struct {
    Text string
}

func (a *CustomError) Error() string {
    return a.Text
}

func foo(fireError bool) error {
    var c *CustomError
    if fireError {
        return &CustomError{Text: "someError"}
    }
    return c
}
```

**Вопрос:** Как скомпилируется программа и что будет выведено? Объясните поведение.

## Решение
Выведется:

```
fire error 1
fire error 2
complete
```

`fire error 2` — вот это неожиданная часть. `foo(false)` не заходит в `if`, значит возвращает `c`, а `c` — это `nil` указатель на `CustomError`. Казалось бы, `err2` должен быть `nil`. Но нет.

Дело в том, как устроен `interface` в Go. Интерфейс (в том числе `error`) — это пара `{type, value}`. Когда `foo` возвращает `c`, происходит упаковка `*CustomError` в интерфейс `error`:

- `type` = `*CustomError` (не nil!)
- `value` = `nil`

Интерфейс считается `nil` только когда **оба** поля nil: `{nil, nil}`. А тут тип задан — `{*CustomError, nil}` — поэтому `err2 != nil` это `true`.

Исправление — возвращать `nil` явно:

```go
func foo(fireError bool) error {
    if fireError {
        return &CustomError{Text: "someError"}
    }
    return nil  // {nil, nil} — настоящий nil интерфейс
}
```

Это классическая ловушка Go — «nil interface vs nil pointer in interface».
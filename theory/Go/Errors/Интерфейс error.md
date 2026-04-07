- `error` — встроенный интерфейс с единственным методом `Error() string` ^error-interface-def
- создание: `errors.New("text")` — простая ошибка; `fmt.Errorf("... %v", arg)` — с форматированием ^error-creation
- ошибку **возвращают последней** из функции — соглашение Go ^error-last
- именование переменных: начинаются с `Err`/`err` (`ErrNotFound`, `errInternal`) ^error-naming-vars
- именование типов: заканчиваются на `Error` (`PathError`, `SyntaxError`) ^error-naming-types
- текст ошибки — **с маленькой буквы** (соглашение гошного коммьюнити) ^error-lowercase

---

## Интерфейс

```go
// builtin
type error interface {
    Error() string
}
```

Любой тип с методом `Error() string` реализует этот интерфейс. ^error-interface-def

## Создание ошибок

```go
// Простая ошибка — достаточно в большинстве случаев
err := errors.New("connection refused")

// С форматированием — когда нужны параметры в тексте
err := fmt.Errorf("user %d not found", userID)
```

На практике чаще всего используют именно эти два способа — отдельные типы нужны редко. ^error-creation

## Соглашения именования

```go
// Переменные — начинаются с Err/err
var ErrNotFound = errors.New("not found")     // публичная
var errInternal = errors.New("internal error") // приватная

// Типы — заканчиваются на Error
type PathError struct { Op, Path string; Err error }
type SyntaxError struct { Msg string; Offset int64 }

// Текст — с маленькой буквы
errors.New("connection refused")  // ✅
errors.New("Connection refused")  // ❌
```

^error-naming-vars
^error-naming-types
^error-lowercase

## Ошибка — последнее возвращаемое значение

```go
// ✅ принято в Go
func ReadFile(path string) ([]byte, error) { ... }

// ❌ непривычно, ломает ожидания
func ReadFile(path string) (error, []byte) { ... }
```

^error-last

## Антипаттерн: лишнее проксирование

```go
// ❌ бессмысленный код — ничего не меняет
func DoSomething() error {
    err := doWork()
    if err != nil {
        return err
    }
    return nil
}

// ✅ эквивалентно и проще
func DoSomething() error {
    return doWork()
}
```

Если сигнатуры совпадают и контекст не нужен — просто проксируй. ^error-proxy-antipattern

## Связь
- [[Способы сигнализации об ошибках]] — error = гошный подход среди прочих
- [[Правила обработки ошибок]] — как правильно обрабатывать и пробрасывать
- [[Оборачивание ошибок]] — fmt.Errorf с %w для wrapping

- multierror — пакет для **списка ошибок**, когда одной недостаточно ^me-what
- сценарий: несколько независимых ветвей могут давать ошибки (обработка среза, параллельные запросы) ^me-scenario
- `Append` для накопления, результат — обычная `error` с методом `Error()` ^me-append
- полностью совместим с `errors.Is`, `errors.As`, `Unwrap` ^me-compat
- компактнее, чем возвращать `(error, error, error)` и проверять каждую ^me-vs-multi-return

---

## Зачем

```go
// Обрабатываем срез — на каждом элементе может быть сбой
func ProcessAll(items []Item) error {
    var result error
    for _, item := range items {
        if err := process(item); err != nil {
            result = multierror.Append(result, err)
        }
    }
    return result  // nil если ошибок не было, иначе список
}
```

Не хотим останавливаться на первой ошибке — обрабатываем весь срез, собираем все ошибки. ^me-append-example

`multierror.Append(nil, ...)` возвращает nil если все переданные ошибки nil. Если хотя бы одна не nil — возвращает multierror. ^me-append-nil

## Совместимость с Is/As/Unwrap

```go
err1 := errors.New("db timeout")
err2 := errors.New("cache miss")
var ErrFatal = errors.New("fatal")

multi := multierror.Append(nil, err1, err2, ErrFatal)

// Оборачивание тоже работает
wrapped := fmt.Errorf("batch failed: %w", multi)

// Is/As проходят по всему списку
fmt.Println(errors.Is(wrapped, ErrFatal))  // true
```

`errors.Is` при поиске проходит по всему списку ошибок внутри multierror. ^me-is-walks-list

## Почему не (error, error, error)

```go
// ❌ загромождает — на каждом уровне проброса проверяй все три
func DoWork() (error, error, error) { ... }

// ✅ одна ошибка — внутри список
func DoWork() error {
    var errs error
    errs = multierror.Append(errs, step1())
    errs = multierror.Append(errs, step2())
    return errs
}
```

^me-why-not-multi-return

## Связь
- [[errors.Is и errors.As]] — multierror совместим с Is/As
- [[Оборачивание ошибок]] — multierror можно оборачивать через %w
- [[Правила обработки ошибок]] — multierror не нарушает правило «обрабатывай один раз»

#flashcards/errors/wrapping

Что такое оборачивание ошибки в Go?
?
![[Оборачивание ошибок#^wrap-def]]

Чем `fmt.Errorf("msg: %v", err)` отличается от `fmt.Errorf("msg: %w", err)` с точки зрения Unwrap?
?
![[Оборачивание ошибок#^wrap-v-vs-w-behavior]]

С какой версии Go появился `%w` в `fmt.Errorf`?
?
![[Оборачивание ошибок#^wrap-percent-w]]

Что вернёт `errors.Unwrap(err)` если err создан через `fmt.Errorf("msg: %v", original)`?
?
![[Оборачивание ошибок#^wrap-percent-v]]

Что делает `errors.Unwrap()`?
?
![[Оборачивание ошибок#^wrap-unwrap-fn]]

Что выведет этот код?
```go
original := errors.New("connection refused")
wrapped := fmt.Errorf("db error: %v", original)
fmt.Println(errors.Unwrap(wrapped))
```
?
`<nil>` — `%v` создаёт строку без метода `Unwrap`, поэтому `errors.Unwrap` возвращает nil. Цепочка разворачивания не работает.
![[Оборачивание ошибок#^wrap-percent-v]]

Что выведет этот код?
```go
err1 := errors.New("disk full")
err2 := fmt.Errorf("write failed: %w", err1)
err3 := fmt.Errorf("save config: %w", err2)
fmt.Println(errors.Unwrap(errors.Unwrap(err3)))
```
?
`disk full` — каждый `%w` добавляет один слой. Первый Unwrap даёт `err2` ("write failed: disk full"), второй — `err1` ("disk full").
![[Оборачивание ошибок#^wrap-chain-behavior]]

Что вернёт `errors.Unwrap` если вызвать его на самой глубокой ошибке в цепочке?
?
![[Оборачивание ошибок#^wrap-chain-behavior]]

Назови два основных сценария использования `%w`.
?
![[Оборачивание ошибок#^wrap-why]]

Как оборачивание помогает при использовании sentinel-ошибок? Почему `errors.Is` продолжает работать?
?
![[Оборачивание ошибок#^wrap-scenario-sentinel]]

Что поддерживает Go 1.20+ относительно `%w` в одном `fmt.Errorf`?
?
![[Оборачивание ошибок#^wrap-go120-multiple]]

Почему `fmt.Errorf("error: %w", "not an error")` — ловушка?
?
![[Оборачивание ошибок#^wrap-percent-w]]

#flashcards/errors/is_as

В чём разница между errors.Is и errors.As? Что каждый проверяет?
?
![[errors_Is_и_errors_As#^is-as-overview]]

Как работает errors.Is внутри? Опиши механизм.
?
![[errors_Is_и_errors_As#^is-mechanism]]

Как работает errors.As внутри? Что происходит при нахождении совпадения?
?
![[errors_Is_и_errors_As#^as-mechanism]]

Почему `wrapped == ErrDB` вернёт false, если ErrDB был обёрнут через `fmt.Errorf("%w", ErrDB)`?
?
![[errors_Is_и_errors_As#^is-mechanism]]

Почему type switch не найдёт исходный тип внутри обёрнутой ошибки?
?
![[errors_Is_и_errors_As#^type-switch-fail]]

Почему нужно всегда использовать errors.Is/As вместо == и type switch?
?
![[errors_Is_и_errors_As#^is-as-why]]

Что выведет этот код?
```go
var ErrDB = errors.New("database problem")
wrapped := fmt.Errorf("query: %w", ErrDB)
fmt.Println(wrapped == ErrDB)
fmt.Println(errors.Is(wrapped, ErrDB))
```
?
`false` затем `true` — `==` сравнивает по указателю/значению интерфейса, wrapped и ErrDB — разные объекты. errors.Is рекурсивно разворачивает цепочку через Unwrap() и находит ErrDB внутри.
![[errors_Is_и_errors_As#^is-mechanism]]

Что выведет этот код?
```go
type DBError struct{ Query string; Err error }
func (e *DBError) Error() string { return e.Err.Error() }

orig := &DBError{Query: "SELECT", Err: errors.New("timeout")}
wrapped := fmt.Errorf("handler: %w", orig)

var dbErr *DBError
fmt.Println(errors.As(wrapped, &dbErr))
fmt.Println(dbErr.Query)
```
?
`true` затем `SELECT` — errors.As разворачивает цепочку, находит `*DBError` и присваивает его в `dbErr`. Поле Query доступно.
![[errors_Is_и_errors_As#^as-mechanism]]

Опиши приём «оборачивание sentinel без контекста». Зачем это делают?
?
![[errors_Is_и_errors_As#^sentinel-wrap-trick]]

Что выведет этот код?
```go
var errBase = errors.New("access denied")
var ErrAccessDenied = fmt.Errorf("%w", errBase)

err := ErrAccessDenied
fmt.Println(err == ErrAccessDenied)
fmt.Println(errors.Is(err, ErrAccessDenied))
```
?
`false` затем `true` — ErrAccessDenied сам является обёрткой над errBase, поэтому == всегда false. errors.Is находит совпадение при разворачивании.
![[errors_Is_и_errors_As#^sentinel-wrap-trick]]

Почему сравнение `err.Error() == "connection refused"` — антипаттерн?
?
![[errors_Is_и_errors_As#^no-string-compare]]

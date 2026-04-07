- `errors.Is` — рекурсивный Unwrap + сравнение по **значению** (для sentinel) ^is-as-overview
- `errors.As` — рекурсивный Unwrap + сравнение по **типу** (для кастомных типов) ^is-as-overview
- **всегда** использовать Is/As вместо == и type switch — код меняется, обёртки добавляются ^is-as-always
- type switch по обёрнутой ошибке **не найдёт** исходный тип (за интерфейсом будет тип обёртки) ^type-switch-fail
- специальный приём: оборачивание sentinel без контекста — чтобы **запретить** == ^sentinel-wrap-trick

---

## errors.Is — сравнение по значению

```go
var ErrDB = errors.New("database problem")

wrapped := fmt.Errorf("query: %w", ErrDB)

// ❌ не сработает — wrapped != ErrDB (разные объекты)
fmt.Println(wrapped == ErrDB)           // false

// ✅ рекурсивно unwrap'ит и находит ErrDB внутри
fmt.Println(errors.Is(wrapped, ErrDB))  // true
```

`errors.Is` рекурсивно вызывает `Unwrap()` на каждом уровне цепочки, пока не найдёт совпадение по значению или не исчерпает цепочку. ^is-mechanism

## errors.As — сравнение по типу

```go
type DatabaseError struct {
    Query string
    Err   error
}
func (e *DatabaseError) Error() string { return e.Err.Error() }

original := &DatabaseError{Query: "SELECT", Err: errors.New("timeout")}
wrapped := fmt.Errorf("handler: %w", original)

// ❌ type switch не найдёт — за интерфейсом тип обёртки, не DatabaseError
switch wrapped.(type) {
case *DatabaseError:  // сюда НЕ зайдёт
}

// ✅ errors.As рекурсивно ищет нужный тип
var dbErr *DatabaseError
if errors.As(wrapped, &dbErr) {
    fmt.Println(dbErr.Query)  // "SELECT"
}
```

`errors.As` рекурсивно разворачивает цепочку, ища ошибку, assignable к типу целевого указателя, и присваивает её при нахождении. ^as-mechanism

## Почему всегда Is/As

```go
// Сейчас работает:
if err == sql.ErrNoRows { ... }

// Завтра разработчик добавил контекст:
return fmt.Errorf("users table: %w", sql.ErrNoRows)

// И ваш == сломался! А errors.Is продолжит работать:
if errors.Is(err, sql.ErrNoRows) { ... }  // ✅ всегда
```

Код меняется — обёртки добавляются. Is/As защищают от этого. ^is-as-why

## Запрет сравнения через ==

```go
// Специально оборачивают без контекста:
var errBase = errors.New("access denied")
var ErrAccessDenied = fmt.Errorf("%w", errBase)

// Теперь == не сработает НИКОГДА — только errors.Is
fmt.Println(err == ErrAccessDenied)           // false
fmt.Println(errors.Is(err, ErrAccessDenied))  // true
```

Приём для библиотек: заставить пользователей сразу использовать errors.Is. ^sentinel-wrap-trick

## Никогда не сравнивай текст ошибки

```go
// ❌ хрупко, антипаттерн
if err.Error() == "connection refused" { ... }
```

Текст ошибки — для людей (логи, экран), не для кода. ^no-string-compare

## Связь
- [[Оборачивание ошибок]] — Is/As работают с цепочкой %w
- [[Sentinel ошибки]] — Is для sentinel, As для кастомных типов
- [[Sentinel vs кастомный тип vs поведение]] — третий способ — по поведению (без Is/As)

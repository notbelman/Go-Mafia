- **sentinel** — когда нужна маркировка без дополнительного контекста (io.EOF, sql.ErrNoRows) ^svb-sentinel-when
- **кастомный тип** — когда нужен контекст: поля с параметрами (os.PathError: Op, Path, Err) ^svb-custom-when
- оба создают **зависимость** между пакетами (нужен импорт для сравнения) ^svb-both-coupling
- **проверка по поведению** — type assertion к анонимному интерфейсу, разрывает зависимость ^svb-behavior-check
- ошибки = **часть публичного API** пакета, относиться бережно ^svb-api

---

## Sentinel — маркировка

```go
var ErrNotFound = errors.New("not found")
var ErrTimeout  = errors.New("timeout")

// Проверка:
if errors.Is(err, ErrNotFound) { ... }
```

Просто, без полей. Достаточно, когда не нужен контекст. ^svb-sentinel-example

## Кастомный тип — контекст

```go
// Пример из stdlib: os.PathError
type PathError struct {
    Op   string  // "open", "read"
    Path string  // "/etc/config"
    Err  error   // underlying error
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}

// Проверка + доступ к полям:
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println(pathErr.Path)  // "/etc/config"
}
```

Неудобно складывать Op, Path в текст ошибки и потом парсить обратно. Проще — отдельный тип. ^svb-custom-example

## Оба создают зависимость

```go
import "myapp/storage"

// Для проверки sentinel — нужен импорт storage
if errors.Is(err, storage.ErrNotFound) { ... }

// Для проверки типа — тоже нужен импорт storage
var dbErr *storage.DatabaseError
if errors.As(err, &dbErr) { ... }
```

^svb-coupling-example

## Проверка по поведению — разрыв зависимости

```go
// Пакет storage — ошибка с поведением
type FSError struct{ path string }
func (e *FSError) Error() string { return "fs: " + e.path }
func (e *FSError) Path() string  { return e.path }

// Другой пакет — НЕ импортирует storage!
func isPathError(err error) bool {
    _, ok := err.(interface{ Path() string })  // анонимный интерфейс
    return ok
}
```

Программист должен знать поведение (какой метод проверять), но коду не нужен импорт. Для исключительных случаев, когда важно разорвать зависимость. ^svb-behavior-example

## Ошибки = публичное API

Публичные sentinel'ы и типы ошибок — такая же часть API, как функции и структуры. Менять их — breaking change. Относиться бережно. ^svb-api-breaking

## Связь
- [[Sentinel ошибки]] — подробнее про sentinel и константные ошибки
- [[errors.Is и errors.As]] — Is для sentinel, As для типов
- [[Правила обработки ошибок]] — когда обрабатывать, когда пробрасывать

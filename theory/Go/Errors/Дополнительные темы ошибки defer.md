- константная ошибка: type definition от string + метод Error() → можно сделать `const`, нельзя перезаписать ^const-error
- проверка по поведению: type assertion к анонимному интерфейсу, **не нужен импорт** пакета с типом ошибки ^behavior-check
- multierror: список ошибок, совместим с `errors.Is`/`errors.As` — проходят по всему списку ^multierror
- стектрейс: стандартный `errors.New` **не даёт**, `github.com/pkg/errors` даёт (~2000 ns vs ~50 ns) ^stacktrace
- явное игнорирование: `_ = f.Close()` — сигнал читателю, что осознанно проигнорировал ^explicit-ignore
- `os.Exit()` — defer'ы **НЕ вызываются**, `runtime.Goexit()` — defer'ы **вызываются** ^exit-vs-goexit

---

## Константные ошибки

```go
// Проблема: sentinel можно перезаписать
var ErrNotFound = errors.New("not found")
// Кто-то может: ErrNotFound = errors.New("hacked!")

// Решение: type definition от string → константа
type ConstError string

func (e ConstError) Error() string {
    return string(e)  // явное приведение, потому что ConstError ≠ string
}

const ErrDatabase ConstError = "database problem"
// ErrDatabase = "other"  // ❌ ошибка компиляции
```

`string(e)` нужен потому что type definition создаёт **новый тип**, не псевдоним. Компилятор не даст вернуть `ConstError` где ожидается `string`. ^const-error-detail

## Проверка по поведению

```go
// С errors.As — нужен импорт пакета:
import "myapp/storage"
var fsErr *storage.FSError
errors.As(err, &fsErr)  // нужен импорт storage

// По поведению — импорт НЕ нужен:
_, ok := err.(interface{ Path() string })  // проверяем только наличие метода
```

Не важно из какого пакета ошибка. Важно только: «есть метод `Path() string`?». Разрывает зависимость между пакетами. Редкий приём. ^behavior-check-detail

## Multierror

```go
func ProcessAll(items []Item) error {
    var result error
    for _, item := range items {
        if err := process(item); err != nil {
            result = multierror.Append(result, err)
        }
    }
    return result  // nil если ошибок не было
}
```

Не останавливаемся на первой ошибке — собираем все. `errors.Is`/`errors.As` проходят по всему списку внутри multierror. ^multierror-detail

## Стектрейс ошибок

```go
// Стандартный — только текст, без стектрейса
err := errors.New("failed")           // ~50 ns/op

// pkg/errors — сохраняет стек вызовов при создании
import pkgerrors "github.com/pkg/errors"
err := pkgerrors.New("failed")        // ~2000 ns/op (в 40 раз дороже)

fmt.Printf("%+v\n", err)
// failed
// main.process
//     /app/main.go:15
// main.main
//     /app/main.go:8
```

Дорого из-за `runtime.Callers`. Не использовать в горячих местах. ^stacktrace-detail

## Явное игнорирование ошибок

```go
// ❌ неявно — забыл или проигнорировал?
f.Close()

// ✅ явно — осознанное решение
_ = f.Close()

// В defer то же самое:
defer func() { _ = f.Close() }()
```

^explicit-ignore-detail

## os.Exit vs runtime.Goexit

|Ситуация|Defer'ы?|Примечание|
|---|---|---|
|Паника (с/без recover)|✅||
|`runtime.Goexit()`|✅|recover вернёт nil (это не паника)|
|`os.Exit()`|❌|процесс убивается мгновенно|
|OOM / stack overflow|❌|fatal error, рантайм завершает процесс|

```go
// os.Exit — defer НЕ вызовется
func main() {
    defer fmt.Println("cleanup")  // ❌ не выполнится
    os.Exit(1)
}

// runtime.Goexit — defer вызовется
go func() {
    defer fmt.Println("cleanup")  // ✅ выполнится
    runtime.Goexit()
}()
```

^exit-vs-goexit-detail

## Связь

- [[Стоимость и аргументы defer]] — стоимость defer, порядок при return
- [[Sentinel ошибки]] — мутабельность sentinel, константные ошибки
- [[errors.Is и errors.As]] — Is для sentinel, As для типов, поведение для разрыва зависимости
- [[Стектрейс ошибки]] — подробнее про pkg/errors
- [[Игнорирование ошибок и ошибки из defer]] — подмена ошибки в defer
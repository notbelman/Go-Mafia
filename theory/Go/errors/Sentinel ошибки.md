- sentinel (дозорная) ошибка — **глобальная переменная** для маркировки конкретной ситуации ^se-def
- нужна, чтобы понять: произошла ли **именно эта** ошибка (sql.ErrNoRows, io.EOF) ^se-purpose
- проблема: глобальную переменную **можно изменить** из другого пакета ^se-mutable-problem
- решение: **константные ошибки** — type definition от string + метод Error() ^se-const-solution
- сравнение через `errors.Is`, **не** через == ^se-compare-is

---

## Sentinel ошибка

```go
// Зарезервированная глобальная ошибка
var ErrDatabaseProblem = errors.New("database problem")

// Использование
func QueryUser(id int) (*User, error) {
    row := db.QueryRow("SELECT ...", id)
    err := row.Scan(&user)
    if err == sql.ErrNoRows {
        return nil, ErrDatabaseProblem
    }
    return &user, err
}
```

`sql.ErrNoRows` — классический пример sentinel: нет строк, запрос пуст. ^se-example

## Проблема: мутабельность

```go
// Кто угодно может поменять глобальную ошибку!
prevEOF := io.EOF
io.EOF = errors.New("hacked!")

fmt.Println(io.EOF == prevEOF)  // false — сломали!
```

Никто так не делает, но технически возможно. ^se-mutable-example

## Решение: константные ошибки

```go
type ConstError string

func (e ConstError) Error() string {
    return string(e)  // type definition → явное приведение
}

const ErrDatabase ConstError = "database problem"

// Нельзя изменить:
// ErrDatabase = "other"  // ❌ ошибка компиляции — константу менять нельзя
```

Хак: type definition от string → можно сделать константой. Метод Error() делает совместимым с интерфейсом error. ^se-const-example

Почему нужно явное приведение `string(e)` в методе Error(): type definition создаёт новый тип, не псевдоним, поэтому прямое использование `e` в строковом контексте запрещено. ^se-const-conversion

## Оборачивание работает

```go
wrapped := fmt.Errorf("query failed: %w", ErrDatabase)
fmt.Println(errors.Is(wrapped, ErrDatabase))  // true
```

ConstError — обычная ошибка, errors.Is/As/Unwrap работают. ^se-wrap-works

## Почему не сравнивать через ==

Если ошибка была обёрнута через `fmt.Errorf("%w", sentinel)`, прямое сравнение `err == sentinel` вернёт false. `errors.Is` раскручивает цепочку обёрток. ^se-why-not-eq

## Связь
- [[errors.Is и errors.As]] — правильное сравнение с sentinel
- [[Оборачивание ошибок]] — sentinel можно оборачивать для маркировки
- [[Sentinel vs кастомный тип vs поведение]] — когда sentinel, когда отдельный тип

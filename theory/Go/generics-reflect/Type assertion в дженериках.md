- **нельзя** делать type switch/assertion напрямую над generic-параметром ^ta-direct-forbidden
- нужно **привести к any** сначала: `any(v).(type)` ^ta-any-cast
- определение типа generic-параметра происходит **в рантайме**, не в compile-time ^ta-runtime
- есть **proposal** для compile-time type switch в будущих версиях Go ^ta-proposal

---

## Проблема: type switch над generic-параметром

```go
func process[T any](v T) {
    switch v.(type) {   // ❌ ошибка компиляции!
    case int:
        fmt.Println("int")
    case string:
        fmt.Println("string")
    }
}
```

`T` — это не интерфейс, а конкретный тип после инстанцирования. Type assertion работает только над интерфейсами. ^ta-why-forbidden

## Решение: привести к any

```go
func process[T any](v T) {
    switch any(v).(type) {  // ✅ приводим к any (пустой интерфейс)
    case int:
        fmt.Println("int")
    case string:
        fmt.Println("string")
    }
}
```

Тоже работает с конкретными constraints:

```go
func process[T int | string](v T) {
    switch any(v).(type) {  // ✅
    case int:
        fmt.Println("int")
    case string:
        fmt.Println("string")
    }
}
```

`any(v)` оборачивает значение в пустой интерфейс, и уже над интерфейсом работает type switch. ^ta-solution

## Парадокс: compile-time → runtime

Дженерики работают на этапе компиляции, но чтобы узнать тип внутри generic-функции — нужен рантайм (type assertion через интерфейс). ^ta-paradox

Есть proposal: `switch type T` — compile-time type switch. Компилятор сам выберет ветку при инстанцировании. Пока не реализовано. ^ta-proposal-detail

## Связь
- [[Constraints]] — constraint ≠ интерфейс для type assertion
- [[Ограничения дженериков]] — другие ограничения
- [[Рефлексия vs интроспекция]] — рефлексия = определение типов в рантайме

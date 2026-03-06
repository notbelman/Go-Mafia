- constraint = **интерфейс**, ограничивающий допустимые типы для generic-параметра ^constraint-def
- типы через `|` на одной строке = **OR** (один из); на новых строках = **AND** (все условия) ^constraint-or-and
- `~int` (тильда) = int **и все type definitions** с базовым типом int ^constraint-tilde
- зарезервированные: `comparable` (==, !=), `any` (любой тип) ^constraint-reserved
- экспериментальные: `constraints.Ordered`, `constraints.Integer`, `constraints.Float` (пакет `golang.org/x/exp/constraints`) ^constraint-experimental

---

## OR — типы на одной строке

```go
type Number interface {
    int | float64 | uint  // ИЛИ: один из этих типов
}
```

Типы через `|` на одной строке означают, что тип должен быть **одним из** перечисленных. ^constraint-or-rule

## AND — условия на разных строках

```go
type MyConstraint interface {
    ~int | ~float64       // базовый тип int или float64
    String() string       // И у типа должен быть метод String()
    fmt.Stringer          // И должен реализовывать Stringer
}
```

Каждая строка — отдельное условие. Тип должен удовлетворять **всем**. ^constraint-and-rule

## Тильда — базовый тип

```go
type Integer1 interface { int | int8 | int16 }     // только эти типы
type Integer2 interface { ~int | ~int8 | ~int16 }   // + type definitions

type MyInt int  // базовый тип = int

func process1[T Integer1](v T) {}
func process2[T Integer2](v T) {}

process1(MyInt(1))  // ❌ MyInt ≠ int
process2(MyInt(1))  // ✅ базовый тип MyInt = int
```

Без тильды принимаются только точные типы. С тильдой — также любой named type поверх базового. ^constraint-tilde-example

## Анонимные constraints (inline)

```go
// Именованный constraint
func sum[T Number](a, b T) T { return a + b }

// Анонимный — прямо в сигнатуре
func sum[T interface{ int | float64 }](a, b T) T { return a + b }

// Сокращённая форма (без interface{})
func sum[T int | float64](a, b T) T { return a + b }
```

Inline constraint удобен для одноразовых ограничений; именованный — если переиспользуется. ^constraint-inline

## comparable и any

```go
func getKeys[K comparable, V any](m map[K]V) []K { ... }
// K — можно ==, != (ключи map)
// V — абсолютно любой тип
```

`comparable` — встроенный constraint: типы, поддерживающие `==` и `!=`. Нужен для ключей map и операций сравнения. ^constraint-comparable

`any` — алиас для `interface{}`, принимает абсолютно любой тип. ^constraint-any

## Связь
- [[Зачем дженерики]] — constraints ограничивают обобщённые типы
- [[Constraints на методы и поля]] — методы в constraints работают, поля нет
- [[Ограничения дженериков]] — что нельзя выразить через constraints

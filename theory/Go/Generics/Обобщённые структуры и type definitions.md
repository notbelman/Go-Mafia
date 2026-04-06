- **структуры** могут быть обобщёнными: `type Set[T comparable] struct { m map[T]struct{} }`
- **методы** обобщённой структуры — в ресивере указывается тип: `func (s *Set[T]) Add(v T)`
- **type definitions** могут быть обобщёнными и частично инстанцированными
- **type aliases** не могут быть обобщёнными (до Go 1.24)
- **variadic + generics** — функция с разным количеством аргументов разных типов при разных вызовах

---

## Обобщённая структура

```go
type Set[T comparable] struct {
    m map[T]struct{}
}

func NewSet[T comparable]() *Set[T] {
    return &Set[T]{m: make(map[T]struct{})}
}

// Метод — тип в ресивере
func (s *Set[T]) Add(v T) {
    s.m[v] = struct{}{}
}

// Если тип не нужен — пропустить
func (s *Set[_]) Len() int {
    return len(s.m)
}

// Использование — инстанцируем один раз
s := NewSet[string]()  // дальше компилятор знает тип
s.Add("hello")
```

Структура параметризована типом `T`. В методах ресивер пишется как `Set[T]`, а не `Set`. Если параметр типа в методе не нужен — можно использовать `_`. ^generic-struct-receiver

## Обобщённые type definitions

```go
type Pair[T, U any] struct { First T; Second U }

// Полное инстанцирование
type StringIntPair = Pair[string, int]

// Частичное — через type definition (не alias!)
type StringPair[U any] Pair[string, U]
// StringPair[int] → Pair[string, int]
```

Частичное инстанцирование возможно только через `type definition`, не через `type alias`. `StringPair[U any]` фиксирует первый параметр как `string`, оставляя `U` свободным. ^generic-typedef-partial

## Type aliases — не обобщённые (до Go 1.24)

```go
type MySlice[T any] []T                 // ✅ type definition
type MyAlias[T any] = []T               // ❌ до Go 1.24
```

С Go 1.24 обобщённые aliases поддерживаются. ^generic-alias-go124

## Прятать сложность за alias/definition

```go
type IntSet = Set[int]           // простой alias для пользователя
type OrderedMap[K comparable, V any] struct { ... }  // внутри сложно

// Для constraints тоже:
type Numeric interface { ~int | ~float64 }
```

Техника: сложный обобщённый тип прячется за простым именем через alias или частичное инстанцирование. ^generic-hide-complexity

## Variadic + generics

```go
func printAll[T any](vals ...T) {
    for _, v := range vals { fmt.Println(v) }
}

printAll(1, 2, 3)           // T=int, 3 аргумента
printAll("a", "b")          // T=string, 2 аргумента
printAll(1.0, 2.0, 3.0, 4.0) // T=float64, 4 аргумента
```

Все аргументы одного типа (variadic), но тип и количество меняются от вызова к вызову. Тип выводится из первого аргумента. ^generic-variadic

## Обобщённые constraints (template template parameters)

```go
type Unsigned interface { ~uint | ~uint8 | ~uint16 }

type Slice[T any] []T

type SliceConstraint[E Unsigned] interface {
    ~[]E  // срез из unsigned-элементов
}

func do[E Unsigned, S SliceConstraint[E]](s S) { ... }
```

Редкий случай — когда нужно достучаться до типа внутри обобщённого контейнера. `S` здесь — constraint, параметризованный другим type-параметром `E`. Аналог template template parameters из C++. ^generic-constraint-template

## Связь
- [[Constraints]] — ограничения на типы-параметры
- [[Type inference и параметры типов]] — для структур всегда явно
- [[Ограничения дженериков]] — нет обобщённых методов, нельзя embed generic-тип

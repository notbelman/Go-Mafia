- **нельзя** создавать методы для: базовых типов, алиасов базовых, указательных типов, интерфейсов ^method-restrictions
- **можно**: type definition от базового → методы для нового типа ^method-typedef-ok
- type definition для функций, map, slice — тоже можно + методы ^method-typedef-funcs
- **unsafe cast** между definition и базовым типом → избежать копирования срезов ^unsafe-cast-slices
- **unions** через unsafe: одна область памяти → разные типы данных (SBO, int64/float64) ^unsafe-unions

---

## Ограничения: где нельзя методы

```go
type AliasInt = int
// func (a AliasInt) Double() int { return int(a) * 2 }
// ❌ нельзя: алиас на базовый тип

type NewInt int
func (n NewInt) Double() int { return int(n) * 2 }
// ✅ можно: type definition

type PtrAlias = *int
type PtrDef *int
// func (p PtrAlias) Method() {}  // ❌ нельзя: указательный тип
// func (p PtrDef) Method() {}    // ❌ нельзя: указательный тип

type MyInterface interface{ Do() }
// func (m MyInterface) Method() {}  // ❌ нельзя: интерфейс
```

^method-restrictions-example

## Definition для функций, map, slice

```go
// Age с методами
type Age int
func (a Age) IsAdult() bool { return a >= 18 }

// Функция с методами
type HandlerFunc func(int) error
func (h HandlerFunc) Filter(x int) bool { return h(x) == nil }

// Множество на основе map
type StringSet map[string]struct{}
func (s StringSet) Has(key string) bool {
    _, ok := s[key]
    return ok
}
func (s StringSet) Add(key string) { s[key] = struct{}{} }
```

^typedef-examples

## Unsafe cast: срезы без копирования

```go
type Integer int

// С копированием (медленно для больших срезов)
func slowCast(src []Integer) []int {
    dst := make([]int, len(src))
    for i, v := range src {
        dst[i] = int(v)
    }
    return dst
}

// Без копирования через unsafe (быстро)
func fastCast(src []Integer) []int {
    return *(*[]int)(unsafe.Pointer(&src))
}
```

Работает потому что `Integer` и `int` имеют **одинаковый underlying type** → идентичный layout в памяти. `unsafe.Pointer` обходит систему типов. ^unsafe-cast-example

## Условие корректности unsafe cast

Unsafe cast между двумя типами корректен только если у них **одинаковый underlying type** — это гарантирует идентичный memory layout. Иначе поведение непредсказуемо. ^unsafe-cast-condition

## Unions через unsafe

```go
type Union struct {
    data [8]byte  // 8 байт — можно трактовать как разные типы
}

func (u *Union) SetInt64(v int64) {
    *(*int64)(unsafe.Pointer(&u.data)) = v
}

func (u *Union) GetInt64() int64 {
    return *(*int64)(unsafe.Pointer(&u.data))
}

func (u *Union) SetFloat64(v float64) {
    *(*float64)(unsafe.Pointer(&u.data)) = v
}

func (u *Union) GetFloat64() float64 {
    return *(*float64)(unsafe.Pointer(&u.data))
}
```

Одна область памяти (8 байт) → int64 или float64. Аналог union из C. Обход системы безопасности типов — использовать осторожно. ^unsafe-union-example

## Связь
- [[Type alias vs Type definition]] — alias vs definition
- [[Методы и ресиверы]] — ресивер = конкретный тип

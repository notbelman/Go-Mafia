- анонимная функция — определяется без имени, в месте использования
- можно присвоить переменной или вызвать сразу `func(x int){ ... }(42)`
- **variadic** (`...T`) — синтаксический сахар над `[]T`
- variadic: только **один**, только **последний** в списке параметров
- nil в variadic = nil slice; `nil...` = распаковка nil slice

---

## Анонимные функции

```go
// Способ 1: присвоить переменной
double := func(x int) int { return x * 2 }
fmt.Println(double(5))  // 10

// Способ 2: вызвать сразу (IIFE)
result := func(a, b int) int {
    return a + b
}(3, 4)
// result = 7 (функция вызвана сразу после определения)
```

Способ 1: `double` — функция. Способ 2: `result` — уже int (результат вызова). ^anon-two-styles

## Variadic параметры

```go
func sum(nums ...int) int {   // nums — это []int
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)       // nums = []int{1, 2, 3}
sum()               // nums = []int{} (пустой slice)
```

Variadic (`...T`) — синтаксический сахар: внутри функции параметр является обычным `[]T`. ^variadic-sugar

## Ограничения variadic

```go
// ❌ variadic не последний
func bad(nums ...int, name string) {}

// ❌ два variadic
func bad2(a ...int, b ...string) {}

// ✅ только один и последний
func ok(prefix string, nums ...int) {}
```

Variadic может быть только один и только последним параметром. ^variadic-constraints

## Variadic и nil

```go
func process(ptrs ...*int) {
    fmt.Println(ptrs)  // что выведет?
}

process(nil)      // [<nil>]     — slice из одного nil-указателя
process(nil...)   // []          — распаковка nil slice = пустой slice

// Потому что variadic = slice:
// nil        → [](*int){nil}      один элемент
// nil...     → ([](*int))(nil)... → ничего
```

`process(nil)` передаёт slice из одного nil-указателя. `process(nil...)` распаковывает nil slice — результат пустой slice. ^variadic-nil

## Связь
- [[Функции первого класса]] — функции как значения
- [[Замыкание]] — анонимная функция + захват переменных
- [[Ограничения функций в Go]] — variadic как обход default params

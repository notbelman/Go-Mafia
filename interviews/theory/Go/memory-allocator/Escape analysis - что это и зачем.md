- escape analysis: компилятор строит **взвешенный граф** и решает — стек или куча, на этапе компиляции
- правила **одинаковы для ВСЕХ типов**: интерфейсы, структуры, примитивы, `new()` — без исключений
- два условия хипа: 
	1) ссылка переживает scope, 
	2) объект слишком большой
		1) `maxStackVarSize` — для обычных переменных (~10MB). 
		2) `maxImplicitStackVarSize` — для reference types, создаваемых через указатель/make (64KB).
- `go build -gcflags="-m" main.go` - посмотреть решение компилятора, есть флаги 
		`-m (подробно)` и `-l (без inlinging)`

---

[[memory Flashcards - escape_analysis]]

Анализ компилятора на этапе компиляции: переменная живёт на стеке или на куче. ^ea-definition

Меньше escapes → меньше аллокаций на куче → реже запускается GC → быстрее. ^ea-why-matters

## Что было бы без escape analysis

Как в C: `return &x` из функции → x жила на стеке → стек разрушился → dangling pointer → segfault. В Go компилятор видит что `&x` уходит наверх и сам переносит x на кучу. Поэтому `return &x` в Go безопасно. ^ea-without-analysis

## Два условия escape на кучу

**1. Компилятор не может доказать**, что на объект не ссылаются после выхода из функции:

```go
func f() *int {
    x := 42
    return &x  // moved to heap: ссылка уходит наверх
}
```
^ea-condition-1-reference

**2. Объект слишком большой** для стека:

```go
var a [10_000_000]int    // moved to heap: > maxStackVarSize (~10MB)
s := make([]int, 70000)  // moved to heap: > maxImplicitStackVarSize (64KB)
```
^ea-condition-2-size

`maxStackVarSize` — для обычных переменных (~10MB). `maxImplicitStackVarSize` — для reference types, создаваемых через указатель/make (64KB). ^ea-size-limits

## Правила одинаковы для ВСЕХ типов

```go
// Интерфейсы НЕ всегда хип
var x interface{} = 42
println(x)         // ✅ стек: println не использует рефлексию

fmt.Println(x)     // ❌ хип: fmt использует рефлексию,
                   //    компилятор не может доказать безопасность
```
^ea-interface-example

```go
// new() НЕ всегда хип
func f() {
    p := new(int)  // ✅ стек: указатель не уходит из функции
    *p = 42
}

func g() *int {
    p := new(int)  // ❌ хип: указатель возвращается наверх
    return p
}
```
^ea-new-example

Неважно: `new`, `make`, литерал, интерфейс — правила escape-анализа идентичны. ^ea-uniform-rules

## Факты о структурах и массивах

Если **одно поле структуры** → хип, вся структура → хип. Если **один элемент массива/среза** → хип, весь массив/срез → хип. Не может часть объекта жить на стеке, а часть в куче. ^ea-struct-array-rule

## Как посмотреть решения компилятора
```bash
go build -gcflags="-m" main.go        # базовый вывод
go build -gcflags="-m -m" main.go     # подробный (почему решил)
go build -gcflags="-m -l" main.go     # -l отключает inlining (чище вывод)


./main.go:5:2: moved to heap: x       # ушла на кучу
./main.go:9:6: x does not escape      # осталась на стеке
```
^ea-flags

## Связь
- [[Что вызывает escape]] — полный список триггеров escape
- [[Inlining и escape]] — inlining может отменить escape
- [[Стек vs Куча]] — почему стек предпочтительнее
- [[Практические приёмы уменьшения аллокаций]] — как помогать escape-анализу

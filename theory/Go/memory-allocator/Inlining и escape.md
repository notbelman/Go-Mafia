- inlining вставляет тело функции в место вызова → может **отменить escape**
- без inlining: `return &x` → хип. С inlining: x оказывается в стеке вызывающей функции
- не инлайнятся: сложные функции, `go`, `defer`, `recover`

---

[[memory Flashcards - inlining_escape]]

Inlining — компилятор вставляет тело функции прямо в место вызова. Это может отменить escape. ^inline-definition

## Как влияет
```go
func newInt() *int {
    x := 42
    return &x  // без inlining: x escapes (указатель уходит наверх)
}

func main() {
    p := newInt() // если newInt() заинлайнится, x окажется
    _ = p         // в стеке main() — escape не нужен
}
```
^inline-escape-example

`gcflags="-m"` покажет:
```
./main.go:3:2: can inline newInt
./main.go:8:13: inlining call to newInt
./main.go:3:2: x does not escape   # остался на стеке main!
```
^inline-compiler-output

Без inlining тот же код:
```
./main.go:3:2: moved to heap: x    # ушёл на кучу
```
^inline-without-output

## Когда функция НЕ инлайнится

Слишком сложная (циклы, switch, много кода), содержит `go`, `defer`, `recover`, вызывает неинлайнящуюся функцию. ^inline-when-not

## На практике

`-gcflags="-m -l"` — отключает inlining, показывает escape решения без этой оптимизации. Полезно когда хочешь увидеть чистую картину что реально escapes. ^inline-flag-l

`//go:noinline` — директива компилятору не инлайнить конкретную функцию (для бенчмарков). ^inline-noinline-directive

## Связь
- [[Escape analysis - что это и зачем]] — что решает escape analysis
- [[Что вызывает escape]] — триггеры escape, которые inlining может отменить

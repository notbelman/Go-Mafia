- Go **не поддерживает** исключения (exceptions) — вместо них явная обработка ошибок ^panic-no-exceptions
- `panic` — механизм для **исключительных** ситуаций, когда продолжение невозможно ^panic-def
- panic ≠ throw: throw делает проблему вызывающей стороны, panic = «не знаю что делать, сдаюсь» ^panic-vs-throw
- `recover` работает **только в defer** — вне defer возвращает nil ^recover-only-defer
- panic принимает `any` — можно передать строку, ошибку, что угодно, даже nil ^panic-any

---

## Panic + recover

```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered:", r)
        }
    }()

    fmt.Println("start")
    panic("something terrible")
    fmt.Println("unreachable")  // никогда не выполнится
}
// start
// recovered: something terrible
```

## Stack unwinding (раскрутка стека)

```
1. panic() → остановка текущей функции
2. вызов defer'ов текущей функции (LIFO)
3. выход → вызов defer'ов вызывающей функции
4. ... и так вверх по стеку
5. Если нашли recover в defer → восстановление
6. Если не нашли → завершение программы с трейсом
```

^panic-unwinding

## Recover только в defer

```go
func process() {
    recover()                    // ❌ бесполезно — вернёт nil
    defer recover()              // ❌ тоже не сработает (не вызов в defer-функции)
    defer func() { recover() }() // ✅ работает
    panic("error")
}
```

`defer recover()` не работает потому что recover вызывается напрямую как аргумент defer, а не внутри анонимной функции — он исполняется сразу, не во время паники. Правильно: `defer func() { recover() }()`. ^recover-defer-gotcha

## Panic принимает any

```go
panic("string error")          // строка
panic(errors.New("real error")) // error
panic(42)                      // int
panic(nil)                     // nil → Go 1.21+ создаёт runtime.PanicNilError
```

^panic-any-examples

## Когда использовать panic

Ошибка программиста (пришло значение, которое **никогда** не должно было прийти). Зависимость не инициализировалась (база не пингуется, без неё приложение бессмысленно). Исключительная ситуация, когда **нечего обрабатывать**. ^panic-when-use

## Почему Go без исключений

Исключения = сложность (гарантии безопасности, утечки ресурсов в C++, неявность потока управления). Go позиционируется как простой язык — явная обработка ошибок проще для понимания. ^panic-why-no-exceptions

## Связь
- [[Defer и паники]] — defer'ы при панике, os.Exit, runtime.Goexit
- [[Восстановимые и невосстановимые ошибки]] — что можно recover, что нельзя
- [[Особенности паник]] — проброс, подмена, panic jump

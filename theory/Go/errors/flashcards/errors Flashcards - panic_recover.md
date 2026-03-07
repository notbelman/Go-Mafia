#flashcards/errors/panic_recover

Почему Go не поддерживает исключения (exceptions)? Что используется вместо?
?
![[Паника и recover#^panic-why-no-exceptions]]

Чем `panic` отличается от `throw` в других языках? В чём семантическая разница?
?
![[Паника и recover#^panic-vs-throw]]

Когда уместно использовать `panic`? Назови три сценария.
?
![[Паника и recover#^panic-when-use]]

Что принимает `panic` в качестве аргумента?
?
![[Паника и recover#^panic-any]]

Опиши механизм stack unwinding при панике — шаги от момента вызова `panic()` до завершения программы (или recover).
?
![[Паника и recover#^panic-unwinding]]

Почему `defer recover()` не работает? Что нужно писать вместо?
?
![[Паника и recover#^recover-defer-gotcha]]

Что выведет этот код?
```go
func process() {
    recover()
    defer recover()
    defer func() { recover() }()
    panic("error")
}
func main() { process() }
```
?
Программа завершится с паникой. Только третий вариант (`defer func() { recover() }()`) работает. Первый вызов до defer бесполезен, второй (`defer recover()`) тоже не работает — recover вызывается напрямую, а не внутри функции.
![[Паника и recover#^recover-defer-gotcha]]

`recover` работает только в defer — что это означает на практике? Что вернёт `recover()` вызванный не в defer?
?
![[Паника и recover#^recover-only-defer]]

Что выведет этот код?
```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered:", r)
        }
    }()
    fmt.Println("start")
    panic("something terrible")
    fmt.Println("unreachable")
}
```
?
```
start
recovered: something terrible
```
`fmt.Println("unreachable")` никогда не выполнится — после `panic` выполнение функции прекращается и начинается раскрутка стека.
![[Паника и recover#^panic-unwinding]]

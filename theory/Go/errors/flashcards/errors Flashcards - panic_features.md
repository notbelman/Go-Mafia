#flashcards/errors/panic_features

Что возвращает `recover()` при `panic(nil)` в Go до 1.21 и с 1.21+? В чём проблема до 1.21?
?
![[Особенности паник#^panic-nil-problem]]

Что выведет этот код на Go 1.21+?
```go
defer func() {
    r := recover()
    fmt.Printf("%T: %v\n", r, r)
}()
panic(nil)
```
?
`*runtime.PanicNilError: panic called with nil argument` — с Go 1.21 `panic(nil)` оборачивается в специальный тип, чтобы отличить от «паники не было».
![[Особенности паник#^panic-nil-121]]

Что такое проброс паники (rethrow) и когда это применяется?
?
![[Особенности паник#^panic-rethrow-usecase]]

Как сделать проброс паники в Go? Приведи паттерн.
?
![[Особенности паник#^panic-rethrow-def]]

Что произойдёт если в `defer` возникнет новая паника, когда уже идёт раскрутка стека от другой паники?
?
![[Особенности паник#^panic-replace-mechanics]]

Что выведет этот код?
```go
func main() {
    defer func() { fmt.Println("recovered:", recover()) }()
    defer func() { panic(3) }()
    defer func() { panic(2) }()
    defer func() { panic(1) }()
    panic(0)
}
```
?
`recovered: 3` — каждая паника в defer заменяет предыдущую. Функция ассоциирована не более чем с одной паникой в любой момент. Последняя (3) — та, что поймает recover.
![[Особенности паник#^panic-replace-mechanics]]

Что такое антипаттерн «panic jump»? Почему это плохо?
?
![[Особенности паник#^panic-jump-def]]

Какие альтернативы panic jump для выхода из вложенных циклов?
?
![[Особенности паник#^panic-jump-alternatives]]

После того как `recover()` перехватил панику — с какого места продолжается выполнение? Что происходит с остальными defer'ами?
?
![[Особенности паник#^panic-after-recover]]

Что выведет этот код?
```go
func example() {
    defer fmt.Println("3: last defer")
    defer func() {
        recover()
        fmt.Println("2: recovered")
    }()
    defer fmt.Println("1: first after panic")
    panic("boom")
}
```
?
```
1: first after panic
2: recovered
3: last defer
```
Defer'ы выполняются LIFO. Recover перехватывает панику на шаге 2, но все три defer всё равно отрабатывают. К месту паники выполнение не возвращается.
![[Особенности паник#^panic-recover-flow]]

#flashcards/errors/defer_panic

Что произойдёт с открытым файлом если функция паникует, а defer f.Close() не написан, но выше по стеку есть recover?
?
![[Defer и паники#^dp-leak-example]]

Почему стоит всегда использовать defer для закрытия ресурсов, даже если сейчас паники нет?
?
![[Defer и паники#^dp-defer-safe]]

Паника долетела до верхушки стека горутины, recover нет — вызовутся ли defer'ы перед крашем?
?
![[Defer и паники#^dp-panic-no-recover-cpp]]

Чем поведение Go при панике без recover отличается от C++ без catch?
?
![[Defer и паники#^dp-panic-no-recover-cpp]]

Что возвращает recover внутри defer при runtime.Goexit()?
?
![[Defer и паники#^dp-goexit-recover-nil]]

Вызываются ли defer'ы при runtime.Goexit()?
?
![[Defer и паники#^dp-goexit-defers]]

Вызываются ли defer'ы при os.Exit(1)?
?
![[Defer и паники#^dp-exit-instant]]

Почему os.Exit опасен в контексте graceful shutdown?
?
![[Defer и паники#^dp-graceful-shutdown]]

Заполни таблицу: какие из четырёх ситуаций вызывают defer'ы — паника+recover, паника без recover, runtime.Goexit(), os.Exit()?
?
![[Defer и паники#^dp-summary-table]]

Что выведет этот код?
```go
func main() {
    defer fmt.Println("cleanup")
    os.Exit(0)
}
```
?
Ничего не выведет — `os.Exit` убивает процесс немедленно, defer'ы не вызываются.
![[Defer и паники#^dp-exit-instant]]

Что выведет этот код?
```go
func main() {
    defer fmt.Println("A")
    panic("boom")
}
```
?
Выведет `A`, затем panic traceback. Defer'ы выполняются даже при панике без recover, перед крашем.
![[Defer и паники#^dp-panic-defers-run]]

Что выведет этот код?
```go
func main() {
    go func() {
        defer fmt.Println("goroutine done")
        runtime.Goexit()
        fmt.Println("unreachable")
    }()
    time.Sleep(time.Millisecond)
}
```
?
Выведет `goroutine done`. `runtime.Goexit()` завершает горутину, но defer'ы вызываются. `recover()` внутри этих defer'ов вернул бы nil, это не паника.
![[Defer и паники#^dp-goexit-recover-nil]]

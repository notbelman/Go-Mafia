#flashcards/errors/ignore_defer

Почему игнорирование ошибки должно быть явным? Что значит явное игнорирование?
?
![[Игнорирование_ошибок_и_ошибки_из_defer#^explicit-ignore]]

Чем `f.Close()` отличается от `_ = f.Close()`? Почему второй вариант лучше?
?
![[Игнорирование_ошибок_и_ошибки_из_defer#^explicit-ignore]]

Как правильно явно игнорировать ошибку внутри defer?
?
![[Игнорирование_ошибок_и_ошибки_из_defer#^defer-explicit-ignore]]

Опиши паттерн подмены ошибки в defer. Когда он применяется?
?
![[Игнорирование_ошибок_и_ошибки_из_defer#^defer-err-replace]]

Почему в паттерне подмены ошибки используют `if err == nil` перед присвоением closeErr?
?
![[Игнорирование_ошибок_и_ошибки_из_defer#^defer-err-replace]]

Что выведет этот код и почему?
```go
func read() (result []byte, err error) {
    err = errors.New("read failed")
    defer func() {
        closeErr := errors.New("close failed")
        if err == nil {
            err = closeErr
        }
    }()
    return nil, err
}
func main() {
    _, err := read()
    fmt.Println(err)
}
```
?
`read failed` — err уже не nil до входа в defer, поэтому `if err == nil` не выполняется и closeErr не подменяет err. Паттерн защищает от перезаписи оригинальной ошибки.
![[Игнорирование_ошибок_и_ошибки_из_defer#^defer-err-replace]]

Почему для телеметрии (логи, метрики) в defer лучше использовать декорирующие обёртки?
?
![[Игнорирование_ошибок_и_ошибки_из_defer#^decorator-pref]]

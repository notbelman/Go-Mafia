#flashcards/errors/interface

Как выглядит интерфейс error в Go? Что нужно реализовать, чтобы удовлетворить его?
?
![[Интерфейс_error#^error-interface-def]]

Какие два основных способа создания ошибок в Go? Когда использовать каждый?
?
![[Интерфейс_error#^error-creation]]

Какое соглашение по позиции error в сигнатуре функции в Go?
?
![[Интерфейс_error#^error-last]]

Каковы соглашения по именованию переменных ошибок (sentinel)? Приведи примеры.
?
![[Интерфейс_error#^error-naming-vars]]

Каковы соглашения по именованию типов ошибок? Приведи примеры.
?
![[Интерфейс_error#^error-naming-types]]

Каким должен быть текст ошибки по соглашению Go: с большой или маленькой буквы? Почему?
?
![[Интерфейс_error#^error-lowercase]]

Что не так с этим кодом?
```go
func DoSomething() error {
    err := doWork()
    if err != nil {
        return err
    }
    return nil
}
```
?
![[Интерфейс_error#^error-proxy-antipattern]]

Что выведет этот код?
```go
type MyError struct{ msg string }
func (e *MyError) Error() string { return e.msg }

var err error = &MyError{"oops"}
fmt.Println(err)
```
?
`oops` — `*MyError` реализует интерфейс error через метод `Error() string`. fmt.Println вызывает Error() для форматирования.
![[Интерфейс_error#^error-interface-def]]

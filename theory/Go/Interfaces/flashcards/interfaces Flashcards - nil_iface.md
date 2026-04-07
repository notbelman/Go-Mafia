#flashcards/interfaces/nil_iface

При каком условии интерфейс == nil?
?
![[nil интерфейса#^nil-iface-both-nil]]

Что происходит с интерфейсом при присваивании типизированного nil указателя?
?
![[nil интерфейса#^nil-typed-nil-not-nil]]

Опиши классическую ошибку с возвратом error из функции. Почему caller видит err != nil?
?
![[nil интерфейса#^nil-classic-error-bug]]

Какое правило позволяет избежать классической ошибки с nil-интерфейсом при возврате error?
?
![[nil интерфейса#^nil-return-explicit-nil]]

Работает ли ловушка с типизированным nil только при return? Где ещё?
?
![[nil интерфейса#^nil-trap-arg-passing]]

Как проверить что data-поле интерфейса nil, когда сам интерфейс не nil?
?
![[nil интерфейса#^nil-reflect-check]]

Что выведет этот код?
```go
func newError() error {
    var err *os.PathError = nil
    return err
}
e := newError()
fmt.Println(e == nil)
```
?
`false` — функция возвращает `(*os.PathError, nil)`, не `(nil, nil)`. Поле tab заполнено типом, поэтому интерфейс не nil.
![[nil интерфейса#^nil-classic-error-bug]]

Что выведет этот код?
```go
func check(v any) {
    fmt.Println(v == nil)
}
var d *int = nil
check(d)
```
?
`false` — при передаче `d` в `any` создаётся eface с `_type = *int` и `data = nil`. Тип поле непустое → не nil.
![[nil интерфейса#^nil-trap-arg-passing]]

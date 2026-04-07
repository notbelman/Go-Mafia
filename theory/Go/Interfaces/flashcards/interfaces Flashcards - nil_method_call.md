#flashcards/interfaces/nil_method_call

Что происходит при вызове метода на nil интерфейсе (nil, nil)? Почему паника?
?
![[Вызов метода на nil#^nil-call-nil-iface-panic]]

Что происходит при вызове метода на интерфейсе (Type, nil)?
?
![[Вызов метода на nil#^nil-call-type-nil-calls]]

nil receiver + обращение к полям: что произойдёт и почему?
?
![[Вызов метода на nil#^nil-receiver-field-panic]]

Что такое nil-safe метод и как он выглядит?
?
![[Вызов метода на nil#^nil-safe-method-idiom]]

Почему fmt.Println(err) не паникует на (Type, nil), а err.Error() паникует?
?
![[Вызов метода на nil#^nil-fmtprintln-catchpanic]]

Что выведет этот код?
```go
type T struct{ val int }
func (t *T) String() string {
    if t == nil { return "nil-T" }
    return fmt.Sprintf("%d", t.val)
}
var t *T
fmt.Println(t)
```
?
`nil-T` — fmt.Println вызывает String() через интерфейс Stringer. t — *T (не nil интерфейс), метод вызывается с nil receiver, проверка срабатывает.
![[Вызов метода на nil#^nil-safe-method-idiom]]

Что выведет этот код?
```go
var w io.Writer
fmt.Println(w == nil)
w.Write([]byte("hi"))
```
?
`true` — первая строка выведет true. Затем паника: w — nil интерфейс (nil, nil), нет itab, метод Write не найти.
![[Вызов метода на nil#^nil-call-nil-iface-panic]]

Что выведет этот код?
```go
type W struct{}
func (w *W) Write(p []byte) (int, error) {
    fmt.Println("called, w==nil:", w == nil)
    return 0, nil
}
var iw io.Writer = (*W)(nil)
iw.Write(nil)
```
?
`called, w==nil: true` — iface имеет непустой tab (*W), метод вызывается. receiver приходит как nil *W. Паники нет — поля не используются.
![[Вызов метода на nil#^nil-call-type-nil-calls]]

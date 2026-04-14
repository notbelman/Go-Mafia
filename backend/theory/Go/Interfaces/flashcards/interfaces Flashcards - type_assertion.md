#flashcards/interfaces/type_assertion

Что произойдёт если сделать `x.(T)` без флага `ok` и тип не совпадёт?
?
![[Type assertion и type switch#^ta-panic-vs-ok]]

Как работает type assertion к конкретному типу под капотом?
?
![[Type assertion и type switch#^ta-under-hood]]

Чем отличается assertion к конкретному типу от assertion к интерфейсу?
?
![[Type assertion и type switch#^ta-concrete-vs-iface-rule]]

Что выведет этот код?
```go
type Fooer interface { Foo() }
type Barer interface { Bar() }
type Thing struct{}
func (t Thing) Foo() {}
func (t Thing) Bar() {}

var x any = Thing{}
switch x.(type) {
case Fooer:
    fmt.Println("Fooer")
case Barer:
    fmt.Println("Barer")
}
```
?
`"Fooer"` — type switch проверяет кейсы по порядку и останавливается на первом совпадении. Thing реализует оба интерфейса, но Fooer стоит первым.
![[Type assertion и type switch#^ts-order-example]]

Почему порядок кейсов в type switch критически важен?
?
![[Type assertion и type switch#^ts-order-caution]]

Что такое анонимный интерфейс в assertion и когда это полезно?
?
![[Type assertion и type switch#^ta-anon-use]]

Покажи синтаксис assertion с анонимным интерфейсом. Что вернёт `ok`?
?
![[Type assertion и type switch#^ta-anon-example]]

Runtime кэширует результаты type switch. Что это означает практически?
?
![[Type assertion и type switch#^ts-cache]]

Что выведет этот код?
```go
var w io.Writer = os.Stdout
f, ok := w.(*os.File)
fmt.Println(ok)
_, ok2 := w.(io.Reader)
fmt.Println(ok2)
```
?
`true` затем `true` — `*os.File` реализует и `*os.File` (конкретный тип, hash совпадает) и `io.Reader` (есть метод Read). Assertion с `ok` никогда не паникует.
![[Type assertion и type switch#^ta-concrete-vs-iface]]

Что произойдёт при `x.(T)` если x содержит nil интерфейс?
?
Паника: `interface conversion: interface is nil, not T`. nil интерфейс не содержит ни типа ни значения, hash сравнить не с чем. Безопасный вариант: `v, ok := x.(T)` вернёт `ok = false`. ^ta-nil-panic
![[Type assertion и type switch#^ta-panic-vs-ok]]

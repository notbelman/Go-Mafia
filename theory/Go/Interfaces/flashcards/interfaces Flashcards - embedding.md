#flashcards/interfaces/embedding

Если встроенный тип реализует интерфейс, что происходит с внешним типом?
?
![[Embedding и реализация интерфейса#^embed-promotes-interface]]

Что происходит если внешний тип определяет метод с тем же именем что и встроенный?
?
![[Embedding и реализация интерфейса#^embed-shadowing]]

Как вызвать перекрытый метод встроенного типа после shadowing?
?
![[Embedding и реализация интерфейса#^embed-shadow-explicit]]

Что выведет этот код?
```go
type User struct{ Name string }
func (u User) Greet() string { return "Hi, " + u.Name }

type Admin struct {
    User
    Role string
}
func (a Admin) Greet() string { return "Admin: " + a.Name }

var g Greeter = Admin{User: User{"Alice"}, Role: "admin"}
fmt.Println(g.Greet())
fmt.Println(g.(Admin).User.Greet())
```
?
`"Admin: Alice"` затем `"Hi, Alice"` — Admin.Greet() перекрывает промоутнутый User.Greet(). Перекрытый метод доступен только через явное обращение `.User.Greet()`.
![[Embedding и реализация интерфейса#^embed-shadow-example]]

Что происходит с одинаковыми методами при встраивании интерфейсов в интерфейс?
?
![[Embedding и реализация интерфейса#^iface-embed-no-conflict]]

Можно ли встраивать struct в интерфейс? Что это даёт?
?
![[Embedding и реализация интерфейса#^struct-in-iface]]

Что происходит с интерфейсом, в который встроен struct, начиная с Go 1.18?
?
![[Embedding и реализация интерфейса#^struct-embed-iface-constraint]]

Почему компилятор считает что Admin реализует Greeter, если Admin не определяет Greet()?
?
![[Embedding и реализация интерфейса#^embed-compiler-sees]]

Что выведет этот код?
```go
type Greeter interface { Hello() string }
type Stringer interface { Hello() string }

type GS interface {
    Greeter
    Stringer
}

type Impl struct{}
func (i Impl) Hello() string { return "hi" }

var x GS = Impl{}
fmt.Println(x.Hello())
```
?
`"hi"` — встроенные интерфейсы с одинаковым методом `Hello()` не конфликтуют и не дублируют метод. Impl реализует GS через один метод Hello().
![[Embedding и реализация интерфейса#^iface-embed-dedup]]

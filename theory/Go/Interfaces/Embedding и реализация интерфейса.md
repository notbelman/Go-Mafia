- если встроенный тип реализует интерфейс → внешний тип тоже (методы **промоутятся**) ^embed-promotes-interface
- если внешний тип определит свой метод с тем же именем → **перекроет** встроенный (shadowing) ^embed-shadowing
- интерфейсы можно **встраивать в интерфейсы**: одинаковые методы не дублируются ^iface-embed-no-dup
- экзотика: можно встроить **struct в интерфейс** — методы struct промоутятся в интерфейс ^struct-in-iface
- с Go 1.18 некоторые интерфейсы — только **constraint** (нельзя создать переменную, подробнее на лекции про дженерики) ^iface-constraint-118

---

## Embedding struct → реализация интерфейса

```go
type Greeter interface { Greet() string }

type User struct{ Name string }
func (u User) Greet() string { return "Hi, " + u.Name }

type Admin struct {
    User  // embedding
    Role string
}

var g Greeter = Admin{User: User{"Alice"}, Role: "admin"}
fmt.Println(g.Greet())  // "Hi, Alice"
```
^embed-struct-example

Admin не определяет `Greet()`, но получает его от User. Компилятор видит промоутнутый метод → интерфейс реализован. ^embed-compiler-sees

## Перекрытие (shadowing)

```go
func (a Admin) Greet() string {
    return "Admin: " + a.Name
}

// Теперь Admin.Greet() перекрыл User.Greet()
fmt.Println(g.Greet())       // "Admin: Alice"
fmt.Println(g.(Admin).User.Greet())  // "Hi, Alice" — явно
```
^embed-shadow-example

Чтобы вызвать перекрытый метод встроенного типа — нужно обратиться явно: `outer.Inner.Method()`. ^embed-shadow-explicit

## Встраивание интерфейсов в интерфейсы

```go
type Greeter interface { Hello() string }
type Stringer interface { Hello() string }

type GreetStringer interface {
    Greeter
    Stringer
}
// Hello() не дублируется — поведение ровно одно
```
^iface-embed-dedup

Одинаковые методы при встраивании не конфликтуют — де-факто один метод. ^iface-embed-no-conflict

## Экзотика: struct встроен в интерфейс

```go
type Printer struct{}
func (p Printer) Print() { fmt.Println("hello") }

type Iface interface {
    Printer  // встроили struct!
}
// Это работает: метод Print() промоутился в интерфейс
```
^struct-embed-iface-example

Крайне редко нужно. С Go 1.18 — это уже **constraint** (нельзя `var x Iface`). ^struct-embed-iface-constraint

## Связь
- [[Встраивание типов]] — механика embedding из лекции про структуры
- [[Что такое интерфейс]] — композиция интерфейсов (Reader + Writer = ReadWriter)
- [[Дженерики vs Интерфейсы]] — constraints = интерфейсы нового вида

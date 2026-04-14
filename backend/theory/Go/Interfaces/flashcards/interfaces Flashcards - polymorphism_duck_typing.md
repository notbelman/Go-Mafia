#flashcards/interfaces/polymorphism_duck_typing

Что такое полиморфизм в контексте Go?
?
![[Полиморфизм и утиная типизация#^poly-def]]

Что такое утиная типизация? Откуда происходит термин?
?
![[Полиморфизм и утиная типизация#^duck-def]]

Сколько методов минимально нужно реализовать, чтобы удовлетворить интерфейс? Можно ли иметь больше методов, чем требует интерфейс?
?
![[Полиморфизм и утиная типизация#^iface-all-methods]]

Что выведет этот код?
```go
type Duck interface { Quack(); Walk() }

type Dog struct{}
func (d Dog) Quack() { fmt.Println("woof") }
func (d Dog) Walk()  { fmt.Println("walk") }

func doStuff(d Duck) { d.Quack() }

doStuff(Dog{})
```
?
`woof` — `Dog` нигде явно не объявляет реализацию `Duck`, но методы совпадают → утиная типизация, компилируется и работает.
![[Полиморфизм и утиная типизация#^duck-example]]

В чём главная проблема утиной типизации в Go? Когда компилятор не поможет?
?
![[Полиморфизм и утиная типизация#^duck-problem]]

Что произойдёт при компиляции этого кода? Найдёт ли компилятор баг?
```go
type UserService interface {
    GetUser(id int) (*User, error)
    UpdateUser(u *User) error
}
type UserRepository interface {
    GetUser(id int) (*User, error)
    UpdateUser(u *User) error
}
func ProvideUserService() UserService {
    return NewUserRepository() // намеренно неверно
}
```
?
Скомпилируется без ошибок — контракты идентичны. Компилятор не видит разницы между `UserService` и `UserRepository`. Баг ловится только в тестах или в проде.
![[Полиморфизм и утиная типизация#^duck-identical-contracts]]

Как полиморфизм реализуется через интерфейсы в Go? Покажи механику на примере.
?
![[Полиморфизм и утиная типизация#^poly-example]]

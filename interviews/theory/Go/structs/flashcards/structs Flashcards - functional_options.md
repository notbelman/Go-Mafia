#flashcards/structs/functional_options

Какую проблему решает паттерн Functional Options? Почему простой конструктор с nil-параметрами плох?
?
![[Functional Options#^fo-problem]]

Что такое `Option` в Functional Options? Какой тип, как создаётся, как применяется?
?
![[Functional Options#^fo-impl]]

Опиши сигнатуру конструктора в паттерне Functional Options.
?
![[Functional Options#^fo-impl]]

Что произойдёт при добавлении новой опции `WithTelegram(tg)` к существующему коду с Functional Options?
?
![[Functional Options#^fo-backward-compat]]

Что такое Configurable Object? Чем отличается от Builder?
?
![[Functional Options#^fo-configurable-object]]

В чём разница между Functional Options и Configurable Object? Какой трейдофф у каждого?
?
![[Functional Options#^fo-comparison]]

Что выведет этот код?
```go
type User struct{ Name, Email string }
type Option func(*User)

func WithEmail(e string) Option {
    return func(u *User) { u.Email = e }
}

opt := WithEmail("a@b.com")
u := &User{Name: "Ivan"}
opt(u)
fmt.Println(u.Name, u.Email)
```
?
`Ivan a@b.com` — `WithEmail` возвращает замыкание, захватывая `e`. При вызове `opt(u)` замыкание устанавливает `u.Email`. `Name` остался без изменений.
![[Functional Options#^fo-impl]]

Что выведет этот код?
```go
type Option func(*User)

func NewUser(name string, opts ...Option) *User {
    u := &User{Name: name}
    for _, opt := range opts {
        opt(u)
    }
    return u
}

u := NewUser("Ivan",
    func(u *User) { u.Email = "first@mail.ru" },
    func(u *User) { u.Email = "second@mail.ru" },
)
fmt.Println(u.Email)
```
?
`second@mail.ru` — опции применяются по порядку, последняя перезаписывает Email. Это следствие того, что конструктор применяет `opt(u)` в цикле последовательно.
![[Functional Options#^fo-impl]]

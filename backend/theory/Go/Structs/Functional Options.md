- `type Option func(*T)` — функция, которая модифицирует объект через указатель
- `WithEmail(e)`, `WithPhone(p)` — замыкают параметр, возвращают Option
- конструктор: обязательные параметры + `...Option`
- **не ломает** существующие вызовы при добавлении новых опций
- альтернатива: **Configurable Object** (fluent setters: `obj.WithName().WithLevel()`)

---

## Проблема: опциональные параметры

```go
// ❌ Плохо: непонятно что к чему, ломается при добавлении полей
func NewUser(name, surname string, email, phone, address *string) *User

u := NewUser("Иван", "Иванов", &email, nil, nil)
// Что значит nil, nil? Непонятно
```

Проблема nil-параметров: при добавлении нового поля в конструктор все существующие вызовы ломаются; `nil` на n-й позиции непонятен без IDE. ^fo-problem

## Functional Options

```go
type User struct {
    Name, Surname        string
    Email, Phone, Address string
}

type Option func(*User)

func WithEmail(email string) Option {
    return func(u *User) { u.Email = email }   // замыкание
}

func WithPhone(phone string) Option {
    return func(u *User) { u.Phone = phone }
}

func NewUser(name, surname string, opts ...Option) *User {
    u := &User{Name: name, Surname: surname}
    for _, opt := range opts {
        opt(u)     // каждая опция модифицирует объект
    }
    return u
}
```

`Option` — это `func(*T)`. `With*`-функции — замыкания, возвращающие `Option`. Конструктор принимает обязательные параметры + `...Option` и применяет каждую опцию к объекту. ^fo-impl

## Использование

```go
// Только обязательные
u1 := NewUser("Иван", "Иванов")

// С email
u2 := NewUser("Иван", "Иванов", WithEmail("ivan@mail.ru"))

// С email и phone
u3 := NewUser("Иван", "Иванов",
    WithEmail("ivan@mail.ru"),
    WithPhone("+79991234567"),
)
```

Добавили `WithTelegram(tg)` → **ни один** существующий вызов не сломался. ^fo-backward-compat

## Альтернатива: Configurable Object

```go
type Logger struct {
    Name  string
    Level string
}

func NewLogger() *Logger { return &Logger{} }

func (l *Logger) WithName(n string) *Logger {
    l.Name = n
    return l   // возвращает себя → цепочка
}

func (l *Logger) WithLevel(lv string) *Logger {
    l.Level = lv
    return l
}

// Использование:
log := NewLogger().WithName("app").WithLevel("INFO")
```

Configurable Object (fluent setters): методы модифицируют объект и возвращают `self` для цепочки вызовов. Похоже на Builder, но строит сам себя — нет отдельного объекта-билдера. ^fo-configurable-object

## Сравнение подходов

Functional Options: опция — внешняя функция-замыкание, не требует методов на типе, легко передавать/хранить/комбинировать. Configurable Object: опция — метод типа, более привычный OOP-стиль, но требует изменения типа при добавлении опций. ^fo-comparison

## Связь
- [[Замыкание]] — WithEmail замыкает параметр
- [[Анонимные функции и variadic]] — `...Option` = variadic
- [[Closer паттерн]] — похожий подход: регистрация функций

- В Go нет наследования. Вместо него — композиция через встраивание (embedding). Встроенные поля/методы "всплывают" наверх, но это синтаксический сахар, не is-a
- Встраивание = has-a с удобным синтаксисом. Нет виртуальных методов, нет super, нет переопределения
- Полиморфизм — через интерфейсы, не через иерархию типов

---

## Встраивание (embedding)

```go
type Logger struct{}
func (l Logger) Log(msg string) { fmt.Println(msg) }

type Server struct {
    Logger          // встраивание — без имени поля
    Host   string
}

s := Server{Host: "localhost"}
s.Log("started")         // вызов "всплыл" — как будто метод Server
s.Logger.Log("started")  // эквивалентно — явный вызов
```

Компилятор **не копирует** методы. Он генерирует обёртку:
```go
// Компилятор неявно создаёт:
func (s Server) Log(msg string) { s.Logger.Log(msg) }
```

## Чем это НЕ наследование

**Нет is-a**: Server не является Logger. Нельзя передать Server туда, где ждут Logger.

```go
func useLogger(l Logger) {}
useLogger(s)         // ОШИБКА компиляции
useLogger(s.Logger)  // ОК — явно достаём встроенное поле
```

**Нет виртуальных методов**: встроенный тип не знает о внешнем.

```go
type Base struct{}
func (b Base) Name() string { return "base" }
func (b Base) Hello() string { return "hello from " + b.Name() }

type Child struct{ Base }
func (c Child) Name() string { return "child" }

c := Child{}
c.Hello()  // "hello from base" — НЕ "hello from child"!
           // Base.Hello() вызывает Base.Name(), не Child.Name()
```

В Java/C++ было бы "hello from child" (виртуальная диспетчеризация). В Go — нет.

**Нет super**: нельзя вызвать "родительскую" версию метода, потому что нет родителя.

## Композиция без встраивания

```go
type Server struct {
    logger Logger   // обычное поле — методы НЕ всплывают
    Host   string
}

s.Log("x")         // ОШИБКА — нет такого метода
s.logger.Log("x")  // ОК — через поле
```

## Когда что использовать

**Встраивание**: когда хочешь "проксировать" интерфейс (io.ReadWriter встраивает Reader + Writer).

**Обычное поле**: когда зависимость — деталь реализации, не часть API.

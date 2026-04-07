- **маленькие интерфейсы**: чем меньше методов, тем больше типов реализуют, тем полезнее ^bp-small-iface
- **accept interfaces, return structs**: принимай интерфейс → гибкость; возвращай конкретный тип → пользователь видит все методы ^bp-accept-return
- **не загрязняй** (interface pollution): интерфейс ради интерфейса — не нужен; нужен при нескольких реализациях, тестах, зависимости между пакетами ^bp-pollution
- «не программируй интерфейсы, **открывай** их» (Rob Pike): сначала конкретный тип, интерфейс только при реальной потребности ^bp-pike-quote
- исключение из «return structs»: вернуть интерфейс, чтобы **гарантировать** соответствие контракту на своей стороне ^bp-return-iface-exception

---

## Маленькие интерфейсы

```go
// ✅ один метод — максимально переиспользуемый
type Reader interface { Read(p []byte) (int, error) }
type Writer interface { Write(p []byte) (int, error) }

// Композиция когда нужно
type ReadWriter interface { Reader; Writer }

// ❌ монстр — сложно реализовать, сложно замокать
type Storage interface {
    Read() error; Write() error; Delete() error
    List() error; Connect() error; Close() error; Ping() error
}
```

Маленький интерфейс легко реализовать и легко замокать в тестах. ^bp-small-iface-why

## Accept interfaces, return structs

```go
// ✅
func Process(r io.Reader) error { ... }  // принимает интерфейс
func NewServer() *Server { ... }          // возвращает конкретный тип

// ❌
func Process(f *os.File) error { ... }    // слишком конкретно
func NewServer() ServerInterface { ... }  // скрывает возможности
```

Почему: принимаем интерфейс → можно передать мок в тестах. ^bp-accept-why

Возвращаем struct → пользователь видит все методы, может использовать как интерфейс или как конкретный тип. ^bp-return-struct-why

## Исключение: вернуть интерфейс

```go
// Когда хотим гарантировать контракт на своей стороне:
func NewReader() io.Reader {
    return &myReader{}  // если myReader не реализует Reader → не скомпилируется
}
```

Возврат интерфейса — compile-time гарантия что тип реализует контракт. ^bp-return-iface-compile

## Interface pollution

```go
// ❌ одна реализация — интерфейс не нужен
type UserGetter interface { GetUser(id int) User }
type userService struct{}
func (u *userService) GetUser(id int) User { ... }

// ✅ интерфейс нужен когда:
// - несколько реализаций
// - нужна абстракция для тестов
// - зависимость между пакетами
```

Три признака когда интерфейс оправдан: несколько реализаций, тесты, межпакетные зависимости. ^bp-when-iface-needed

## «Открывай интерфейсы» — Rob Pike

В Java/C++: начинают с абстрактных классов и интерфейсов → потом реализации. ^bp-java-approach

В Go **наоборот**: сначала конкретный тип (данные, методы) → интерфейс только когда появляется реальная потребность абстрагировать. ^bp-go-approach

## Связь
- [[Расположение интерфейсов]] — где объявлять: producer vs consumer
- [[Полиморфизм и утиная типизация]] — почему маленькие интерфейсы снижают риск путаницы
- [[Что такое интерфейс]] — interface guard для гарантии контракта

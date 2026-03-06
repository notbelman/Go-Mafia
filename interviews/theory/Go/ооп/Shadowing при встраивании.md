- **Shadowing**: если внешняя структура объявляет поле/метод с тем же именем что у встроенной — внешнее "затеняет" внутреннее
- Затенённое поле/метод не удаляется — доступно через явное имя встроенного типа
- При двух встроенных типах с одинаковым методом — ambiguous, компилятор требует явный вызов

---

## Shadowing поля

```go
type Base struct {
    Name string
}

type Extended struct {
    Base
    Name string  // затеняет Base.Name
}

e := Extended{
    Base: Base{Name: "base"},
    Name: "extended",
}

fmt.Println(e.Name)      // "extended" — внешнее поле
fmt.Println(e.Base.Name) // "base"     — всё ещё доступно
```

## Shadowing метода

```go
type Logger struct{}
func (Logger) Log() { fmt.Println("Logger.Log") }

type App struct{ Logger }
func (App) Log() { fmt.Println("App.Log") }  // затеняет Logger.Log

a := App{}
a.Log()        // "App.Log"
a.Logger.Log() // "Logger.Log" — явный доступ
```

Это НЕ переопределение (override). Base не знает о затенении, виртуальной диспетчеризации нет.

## Конфликт двух встроенных типов

```go
type A struct{}
func (A) Hello() { fmt.Println("A") }

type B struct{}
func (B) Hello() { fmt.Println("B") }

type C struct {
    A
    B
}

c := C{}
c.Hello()    // ОШИБКА компиляции: ambiguous selector c.Hello
c.A.Hello()  // ОК — "A"
c.B.Hello()  // ОК — "B"
```

**Решение конфликта**: либо явный вызов, либо определи свой метод Hello на C (он затенит оба).

```go
func (c C) Hello() { c.A.Hello() }  // явный выбор
```

## Правило приоритета

Компилятор ищет метод/поле по "глубине": сначала на самом типе, потом на встроенных типах глубины 1, потом глубины 2 и т.д. Одинаковая глубина + одинаковое имя = ошибка компиляции.

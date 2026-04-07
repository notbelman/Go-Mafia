- `x.(T)` — если T конкретный тип: сравнение hash типов; если T интерфейс: проверка реализации ^ta-mechanism
- без булева флага → **паника** если не совпало; с флагом `v, ok := x.(T)` → безопасно ^ta-panic-vs-ok
- type switch: `switch v := x.(type)` — проверка нескольких типов; **порядок кейсов важен** — первый совпавший побеждает ^ts-order
- анонимные интерфейсы в assertion: `x.(interface{ Bar() })` — проверка без именованного интерфейса ^ta-anonymous-iface
- runtime кэширует результаты type switch — повторные проверки быстрые ^ts-cache

---

## Type assertion

```go
var w io.Writer = os.Stdout

f := w.(*os.File)           // паника если не *os.File
f, ok := w.(*os.File)       // ok = false без паники
```
^ta-example

Под капотом: сравнивается `tab._type.hash` с hash запрашиваемого типа. Совпало → возвращаем `data`. Нет → паника или `ok = false`. ^ta-under-hood

## Assertion к интерфейсу vs конкретному типу

```go
var a any = os.Stdout

// К конкретному типу — сравнение hash
f := a.(*os.File)

// К интерфейсу — runtime проверяет: реализует ли *os.File метод Read?
r := a.(io.Reader)  // ✅ реализует
```
^ta-concrete-vs-iface

Assertion к конкретному типу → сравнение hash. Assertion к интерфейсу → runtime проверяет реализацию методов. ^ta-concrete-vs-iface-rule

## Type switch — порядок важен!

```go
type Fooer interface { Foo() }
type Barer interface { Bar() }
type FooBarer interface { Foo(); Bar() }

type Thing struct{}
func (t Thing) Foo() {}
func (t Thing) Bar() {}

var x any = Thing{}

switch x.(type) {
case Fooer:       // ← Thing реализует Fooer → попадёт СЮДА
    fmt.Println("Fooer")
case Barer:       // тоже реализует, но сюда не дойдёт
    fmt.Println("Barer")
case FooBarer:    // тоже реализует, но сюда не дойдёт
    fmt.Println("FooBarer")
}
// Выведет: "Fooer"
```
^ts-order-example

Если `Fooer` и `Barer` поменять местами — выведет `"Barer"`. Первый совпавший кейс побеждает. Будьте осторожны с порядком. ^ts-order-caution

## Анонимный интерфейс в assertion

```go
type Fin struct{}
func (f Fin) Foo() {}
func (f Fin) Bar() {}

var x Fooer = Fin{}

// Проверяем наличие Bar() без создания отдельного интерфейса
v, ok := x.(interface{ Bar() })  // ok = true
```
^ta-anon-example

Удобно для разовых проверок — не нужно плодить именованные интерфейсы. ^ta-anon-use

## Связь
- [[Статический и динамический тип]] — assertion проверяет динамический тип
- [[iface структура]] — hash в itab для быстрого сравнения
- [[Копирование и ловушки с типами]] — одинаковое имя метода, разные сигнатуры → assertion падает

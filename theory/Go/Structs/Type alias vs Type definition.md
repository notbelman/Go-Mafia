- **type alias** (`type A = int`) — синоним, **тот же тип**, каст не нужен, методы базового доступны
- **type definition** (`type A int`) — **новый тип**, каст нужен, методы НЕ наследуются
- definition: поля структуры **видны** (layout памяти), методы **нет** (ресивер — другой тип)
- каждый definition + его базовый тип имеют один и тот же **underlying type**

---

## Синтаксис

```go
type MyAlias    = int   // alias: знак "=" → синоним
type MyNewType    int   // definition: без "=" → новый тип
```

`=` → alias (синоним), без `=` → definition (новый тип). ^alias-syntax

## Присваивание

```go
type AliasInt = int
type NewInt     int

var x int = 42

var a AliasInt = x    // ✅ тот же тип — каст не нужен
var b NewInt   = x    // ❌ cannot use x (int) as NewInt
var c NewInt   = NewInt(x)  // ✅ явный каст
```

Alias присваивается без каста — это один и тот же тип. Definition требует явного каста. ^alias-assignment

## Методы

```go
type Data struct{ Value int }
func (d Data) Print() { fmt.Println(d.Value) }

type AliasData = Data     // alias
type NewData     Data     // definition

ad := AliasData{Value: 1}
ad.Print()    // ✅ alias → тот же тип → метод доступен

nd := NewData{Value: 2}
nd.Print()    // ❌ новый тип → метод НЕ наследуется
nd.Value      // ✅ поля видны (это layout памяти, не семантика)
```

**Почему поля видны, а методы нет?** Метод = функция с ресивером. Ресивер Print — `Data`, а не `NewData`. Это разные типы → метод не подходит. ^alias-methods-why

Поля — просто расположение данных в памяти, одинаковое для обоих. ^alias-fields-visible

## Цепочка определений

```go
type A int        // underlying type = int
type B A          // underlying type = int (тот же!)
type C = A        // alias для A, underlying type = int
```

`A`, `B` и `int` — разные типы, но **один underlying type**. ^alias-underlying-chain

Underlying type определяется рекурсивно: если основой служит другой named type — берём его underlying type. Alias не создаёт новый underlying type — это тот же тип. ^alias-underlying-rule

## Использование alias на практике

Type alias используется для: постепенного рефакторинга (переименование типа без breaking change), кросс-пакетного экспорта типа под другим именем. ^alias-use-cases

## Связь
- [[Ограничения методов и unsafe cast]] — когда можно создавать методы
- [[Структуры основы]] — структуры как базовый тип для definition
- [[Методы и ресиверы]] — ресивер = конкретный тип

- **интроспекция** — исследовать тип/свойства объекта в рантайме (только чтение)
- **рефлексия** — исследовать **и модифицировать** структуру и поведение в рантайме
- рефлексия = **метапрограммирование в рантайме** (дженерики = в compile-time)
- два базовых типа: `reflect.Type` (TypeOf) и `reflect.Value` (ValueOf)
- **Kind** vs **Type**: kind = базовый тип (int), type = конкретный (MyInt). Для alias: kind == type

---

## Интроспекция vs рефлексия

```
Интроспекция: «что это за объект? какие у него поля? методы?»
Рефлексия:    «что это за объект? + давай поменяем ему поля и вызовем методы»
```

Go поддерживает полную рефлексию через пакет `reflect`.

## reflect.Type и reflect.Value

```go
var x float64 = 3.14

t := reflect.TypeOf(x)   // reflect.Type — информация о типе
v := reflect.ValueOf(x)  // reflect.Value — информация о значении

fmt.Println(t)            // float64
fmt.Println(v)            // 3.14
fmt.Println(v.Type())     // float64
```

Под капотом: TypeOf и ValueOf принимают `any` → неявное преобразование в пустой интерфейс → анализ двух указателей интерфейса (itab + data).

## Kind vs Type

```go
type MyInt int

var a int = 42
var b MyInt = 42

// alias (type MyAlias = int)
ta := reflect.TypeOf(a)
fmt.Println(ta.Name(), ta.Kind())  // int, int  — совпадают

// type definition (type MyInt int)
tb := reflect.TypeOf(b)
fmt.Println(tb.Name(), tb.Kind())  // MyInt, int — различаются!
```

Kind возвращает **базовый** тип (enum: int, string, struct, ptr, slice...). Type возвращает **конкретный** тип. Для type definition: Kind = базовый, Type = определённый.

## Kind для проверок в коде

```go
v := reflect.ValueOf(x)

switch v.Kind() {
case reflect.Int:
    fmt.Println("integer:", v.Int())
case reflect.Float64:
    fmt.Println("float:", v.Float())
case reflect.String:
    fmt.Println("string:", v.String())
}
```

## API: самый большой тип

```go
var x uint8 = 42
v := reflect.ValueOf(x)

// Возвращает uint64 (самый большой), не uint8
val := v.Uint()  // uint64
```

Для простоты API: getter-методы Value возвращают максимальный тип семейства (Int() → int64, Uint() → uint64, Float() → float64). Информация о реальном типе сохраняется в Kind/Type.

## Связь
- [[Три свойства рефлексии]] — как работает преобразование interface ↔ reflect
- [[Зачем дженерики]] — дженерики = compile-time, рефлексия = runtime
- [[Возможности рефлексии]] — что можно делать с reflect

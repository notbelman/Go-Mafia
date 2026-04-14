- пустой интерфейс `interface{}` / `any` — **отдельная структура** `eface`, проще чем `iface`
- вместо itab хранит только `_type` — нет методов, таблица не нужна
- `_type` лежит прямо в eface → **быстрее type switch** (нет лишнего разыменования через itab)
- boxing скаляров (int, bool) в интерфейс → копия уходит в heap (аллокация)
- указатели при boxing кладутся **напрямую** в `data` (оптимизация — без лишней аллокации)
- `var a any // {тип: nil, значение: nil}`

---

## Структура в runtime

```go
// runtime/runtime2.go
type eface struct {
    _type *_type         // только тип, без методов
    data  unsafe.Pointer // указатель на значение
}
```

`eface` — отдельная структура от `iface`, используется для пустого интерфейса `interface{}` / `any`. ^eface-separate-struct

Вместо `itab` хранит только `_type` — методов нет, таблица не нужна. ^eface-no-itab

## Визуально

```
var a any = 42

a (eface, 16 байт)
+--------+--------+
| _type  |  data  |
+---+----+----+---+
    |         |
    v         v
  *_type     int(42)
+----------+
| size: 8  |
| kind: int|
| hash: ...|
+----------+
```

## Почему отдельная структура?

```
iface (с методами)            eface (пустой)
+-------+-------+            +-------+-------+
|  tab  | data  |            | _type | data  |
+---+---+-------+            +-------+-------+
    |
    v
  itab
+-------+
| _type | ← тип тут, лишний pointer dereference
+-------+
```

В `iface` чтобы добраться до типа надо: `tab` → `itab._type` — лишнее разыменование. В `eface` `_type` лежит прямо в структуре. ^eface-type-direct

Это даёт два преимущества: **экономия памяти** (не нужен itab) + **быстрее type switch** (`_type` прямо в eface, не надо разыменовывать tab). ^eface-advantages

## Boxing (упаковка в интерфейс)

```go
var a any = 42   // int копируется в heap, eface.data указывает туда
var b any = &x   // указатель кладётся напрямую в eface.data (оптимизация)
```

**Скаляры** (int, bool и т.д.) при упаковке в интерфейс → **аллокация в heap**, копия значения. ^eface-boxing-scalar-alloc

**Указатели** при упаковке → кладутся **напрямую** в `eface.data`, без лишней аллокации. ^eface-boxing-ptr-no-alloc

## Связь
- [[iface структура]] — интерфейс с методами: itab + data
- [[any vs interface{}]] — any = алиас interface{}, обе → eface
- [[nil интерфейса]] — eface с nil _type и nil data → nil

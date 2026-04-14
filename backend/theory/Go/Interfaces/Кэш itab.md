- itab кэшируется в **глобальной hash-таблице** в runtime (`itabTable`) ^itab-cache-global
- ключ = пара (interfacetype, _type), значение = готовый *itab ^itab-cache-key
- при первом присваивании: runtime ищет itab в таблице → не нашёл → **вычисляет** (сопоставляет методы) → кладёт в кэш ^itab-cache-first
- повторное присваивание той же пары → **O(1) lookup** из кэша, без пересчёта ^itab-cache-repeat
- вычисление itab = проход по методам интерфейса и типа (оба отсортированы) → **O(n+m)**, где n и m — кол-во методов ^itab-calc-complexity

---

## Структура кэша

```go
// runtime/iface.go (упрощённо)
type itabTableType struct {
    size    uintptr         // размер таблицы (степень двойки)
    count   uintptr         // кол-во записей
    entries []*itab         // hash-таблица с open addressing
}
```

Кэш реализован как hash-таблица с open addressing. ^itab-cache-structure

## Как происходит lookup

```
var w io.Writer = myFile{}  // первый раз

1. hash = hash(io.Writer) ^ hash(*myFile)
2. ищем в itabTable по hash → не нашли
3. вычисляем itab: сопоставляем методы интерфейса и типа
   (оба списка отсортированы → один проход O(n+m))
4. кладём в itabTable
5. возвращаем *itab

var w2 io.Writer = myFile{}  // второй раз
1. hash = hash(io.Writer) ^ hash(*myFile)
2. ищем в itabTable → нашли → возвращаем *itab (O(1))
```

Ключ поиска в кэше: `hash(interfacetype) ^ hash(_type)`. ^itab-cache-hash

## Когда itab НЕ кэшируется

Если тип **не реализует** интерфейс — runtime тоже кэширует результат (с пометкой `fun[0] = 0`), чтобы повторный type assertion не пересчитывал заново. ^itab-cache-fail

## Связь
- [[iface структура]] — itab = tab поле в iface
- [[Диспетчеризация и девиртуализация]] — кэш ускоряет повторные присваивания

- assertion к **конкретному типу** `x.(T)`: сравнение hash → **O(1)**, очень быстро ^ta-concrete-o1
- assertion к **интерфейсу** `x.(I)`: runtime проверяет реализацию всех методов → **дороже** ^ta-iface-expensive
- конкретный тип: один `if` — совпал hash или нет ^ta-concrete-mechanism
- интерфейс: нужно найти/вычислить itab для пары (I, динамический тип) — возможно O(n+m) ^ta-iface-mechanism
- на практике: itab кэшируется → повторные assertion к интерфейсу тоже быстрые, но **первый** вызов дороже ^ta-cache-practical

---

## Assertion к конкретному типу — O(1)

```go
var x any = 42
v, ok := x.(int)

// Под капотом:
// if x._type.hash == int.hash → ok, return data
// else → !ok
```

Одно сравнение hash — мгновенно. ^ta-concrete-impl

## Assertion к интерфейсу — дороже

```go
var x any = os.Stdout
r, ok := x.(io.Reader)

// Под капотом:
// 1. ищем itab для пары (io.Reader, *os.File) в кэше
// 2. нашли → проверяем fun[0] != 0 → ok
// 3. не нашли → вычисляем itab (O(n+m)) → кэшируем → ok/!ok
```

Первый вызов: вычисление + кэширование. ^ta-iface-first-call

Повторные: lookup из кэша. ^ta-iface-repeated

## Type switch — кэширование

```go
switch x.(type) {
case int:       // hash сравнение — O(1)
case io.Reader: // itab lookup — кэшируется
}
```

Runtime кэширует результаты type switch — повторные итерации цикла с тем же типом быстрые. ^ta-switch-cache

## Сравнение

| | К конкретному типу | К интерфейсу |
|---|---|---|
| Механизм | сравнение hash | поиск/вычисление itab |
| Первый вызов | O(1) | O(n+m) методов |
| Повторный | O(1) | O(1) из кэша |
| Когда использовать | знаешь точный тип | проверяешь поведение |

## Связь
- [[Кэш itab]] — itab кэшируется глобально
- [[Type assertion и type switch]] — синтаксис и порядок кейсов
- [[Диспетчеризация и девиртуализация]] — девиртуализация через assertion к конкретному типу = O(1)

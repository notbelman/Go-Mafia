**package** — полное имя сервиса: `order.v1.OrderService`. Сменил package — клиент не найдёт сервис.

**Что генерируется в Go:**
- `enum` → `int32` + константы
- `repeated` → `[]T`
- `optional` → `*T` (pointer, можно отличить "не передали" от zero value)
- `map` → `map[K]V`
- `message` → struct
- `service` → interface + клиент
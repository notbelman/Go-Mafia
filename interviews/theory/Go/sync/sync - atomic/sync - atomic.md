Атомарные операции — выполняются целиком за одну CPU инструкцию. Другой поток не увидит промежуточное состояние. ^atomic-definition

|Метод|Что делает|
|---|---|
|`Load()`|Атомарное чтение|
|`Store(val)`|Атомарная запись|
|`Add(delta)`|Сложение, возвращает новое значение|
|`Swap(new)`|Обмен, возвращает старое|
|`CompareAndSwap(old, new)`|Записать new только если текущее == old|

^atomic-methods-table

**Типы (Go 1.19+):** `atomic.Int32`, `Int64`, `Uint32`, `Uint64`, `Bool`, `Pointer[T]`, `Value` ^atomic-types-go119

## Связь
- [[Atomic vs Mutex]] — когда atomic, когда mutex
- [[atomic.Value]] — хранение произвольных типов
- [[Почему НЕ atomic везде]] — ограничения atomic
- [[CAS и CAS loop]] — ключевая атомарная операция

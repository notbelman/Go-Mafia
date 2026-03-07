- RCU (Read-Copy-Update) — атомарная подмена указателя на целую структуру данных. Читатели работают с копией, писатель создаёт новую и подменяет указатель ^rcu-def
- Работает когда операции — только чтение, а запись = полная замена (не модификация) ^rcu-when
- Нет мьютексов, нет блокировок: только atomic.Store/Load на указателе ^rcu-no-mutex

---

## Идея

Есть кэш (мапа), который обновляется раз в минуту из внешнего хранилища. Между обновлениями — только чтение.

**С мьютексом:**
```go
type Cache struct {
    mu   sync.Mutex
    data map[string]string
}

func (c *Cache) Get(key string) string {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.data[key]
}
```

**С RCU (без мьютекса):**
```go
type Cache struct {
    data unsafe.Pointer  // *map[string]string
}

func (c *Cache) Get(key string) string {
    m := (*map[string]string)(atomic.LoadPointer(&c.data))
    return (*m)[key]
}

func (c *Cache) sync() {
    newMap := loadFromStorage()           // загрузка без блокировок
    p := unsafe.Pointer(&newMap)
    atomic.StorePointer(&c.data, p)       // атомарная подмена
}
```

^rcu-code

## Как работает

```
Горутина A: Load → получила старую мапу → работает с ней
                   ↓ (в это время)
Синхронизатор: Store → подменил указатель на новую мапу
                   ↓
Горутина B: Load → получила НОВУЮ мапу → работает с ней
```

A и B работают с **разными** копиями одновременно. Это нормально — данные обновляются раз в минуту, секундное расхождение некритично. ^rcu-concurrent-copies

Когда A закончит — следующий Load вернёт уже новую мапу. ^rcu-eventual

## Когда подходит

- Кэш с периодическим обновлением (read-heavy, write-rare) ^rcu-use-cache
- Конфигурация, загружаемая из внешнего источника ^rcu-use-config
- Любая структура где запись = полная замена, не модификация отдельных полей ^rcu-use-replace

**Не подходит**: если нужно добавлять/удалять отдельные элементы конкурентно. ^rcu-not-for-partial

## Связь
- [[CAS паттерны]] — RCU использует atomic Store/Load
- [[Happens-before через atomic]] — atomic.Store гарантирует видимость новых данных

| Критерий            | sync.Map   | map + RWMutex |
| ------------------- | ---------- | ------------- |
| Read-heavy          | ✓ быстрее  | медленнее     |
| Write-heavy         | медленнее  | ✓ быстрее     |
| Range/итерация      | медленно   | ✓ быстрее     |
| len()               | нет        | ✓ O(1)        |
| Типобезопасность    | нет (any)  | ✓ есть        |
| Много горутин (50+) | ✓ лучше    | contention    |
| Мало горутин (<10)  | overhead   | ✓ проще       |
| Disjoint keys       | ✓ идеально | норм          |
^vs-comparison-table

По умолчанию: map + RWMutex
sync.Map: только read-heavy + много горутин + замерил ^vs-default-rule

## Главное отличие: где происходит "работа"

**map + RWMutex:**
```
RLock()  →  читаем map  →  RUnlock()
   ↑                           ↑
   └── каждый раз atomic.Add ──┘
```
Каждое чтение модифицирует `readerCount` → cache contention на многих ядрах. ^vs-rwmutex-contention

**sync.Map:**
```
Load()  →  atomic.Load(read)  →  ищем в read map  →  нашли? return!
                                      ↓ не нашли
                              берём mutex, ищем в dirty
```
Если ключ в `read` — **вообще никаких локов и атомиков на запись**.
Просто атомарное чтение указателя и поиск в обычной map. ^vs-syncmap-lockfree

---

## Когда sync.Map быстрее

sync.Map оптимизирован для **двух конкретных сценариев** (это из официальной документации): ^vs-two-scenarios

### Сценарий 1: Write-once, read-many (кэш который только растёт)
```go
// Ключ записывается ОДИН раз, читается МНОГО раз
var cache sync.Map

func GetUser(id int) User {
    // 99% случаев — найдём в read map, без локов
    if val, ok := cache.Load(id); ok {
        return val.(User)
    }
    
    // 1% случаев — грузим из БД, записываем
    user := loadFromDB(id)
    cache.Store(id, user)  // медленно, но редко
    return user
}
```
^vs-scenario-write-once

**Почему быстро:** после первой записи ключ "промоутится" в read map.
Все последующие чтения — lock-free. ^vs-scenario-write-once-why

### Сценарий 2: Disjoint keys (разные горутины работают с разными ключами)
```go
// Горутина 1 работает с ключами "user:1", "user:2", ...
// Горутина 2 работает с ключами "order:1", "order:2", ...
// Они НЕ пересекаются

var data sync.Map

func Worker1() {
    data.Store("user:1", value1)  // свои ключи
    data.Load("user:2")
}

func Worker2() {
    data.Store("order:1", value2)  // свои ключи
    data.Load("order:2")
}
```
^vs-scenario-disjoint

**Почему быстро:** каждая горутина модифицирует свои entry.
Нет contention на одних и тех же данных. ^vs-scenario-disjoint-why

---

## Когда map + RWMutex быстрее

### Частые записи (write-heavy)
```
Бенчмарк (10 горутин, 100k записей):
──────────────────────────────────────
RWMutex + map     113ms   ← быстрее!
sync.Map          203ms   ← почти 2x медленнее
```
^vs-bench-write-10

**Почему sync.Map медленный на записях:** ^vs-write-heavy-why
1. Store() всегда берёт mutex
2. Если ключа нет в read — копирует весь read в dirty
3. После N промахов — промоутит dirty в read (копирует всё обратно)

Эти копирования — дорого. ^vs-write-heavy-copies

### Range/итерация по всем ключам
```
Бенчмарк (10 горутин, 100 итераций по всей map):
────────────────────────────────────────────────
RWMutex + map     143ms   ← в 6.5x быстрее!
sync.Map          949ms
```
^vs-bench-range-10

**Почему:** `sync.Map.Range()` не атомарен и делает много работы внутри.
`RWMutex` просто лочит один раз и итерирует обычную map. ^vs-range-why

### Частые удаления

sync.Map не удаляет сразу — помечает как `expunged`.
Реальная очистка происходит при следующем промоуте dirty→read. ^vs-delete-lazy

---

## Бенчмарки из реального репозитория

**10 горутин:**
| Операция | RWMutex | sync.Map | Быстрее |
|:---------|:--------|:---------|:--------|
| Write 100k | 113ms | 203ms | RWMutex 1.8x |
| Read 100k | 31ms | 38ms | RWMutex 1.2x |
| Range 100 | 143ms | 949ms | RWMutex 6.6x |
^vs-bench-10goroutines

**100 горутин:**
| Операция | RWMutex | sync.Map | Быстрее |
|:---------|:--------|:---------|:--------|
| Write 100k | 1680ms | 261ms | sync.Map 6.4x |
| Read 100k | 280ms | 120ms | sync.Map 2.3x |
| Range 10 | 96ms | 836ms | RWMutex 8.7x |
^vs-bench-100goroutines

**Вывод:** sync.Map выигрывает только при высокой конкурентности (много горутин) и read-heavy нагрузке. ^vs-bench-conclusion

---

## Другие минусы sync.Map

### Нет типобезопасности
```go
// sync.Map
val, _ := m.Load("key")
user := val.(User)  // type assertion, может паниковать

// map + RWMutex
user := m["key"]  // компилятор проверит тип
```
^vs-no-typesafety

### Нет len()
```go
// sync.Map — нельзя узнать размер без итерации
count := 0
m.Range(func(k, v any) bool {
    count++
    return true
})

// map + RWMutex
len(m)  // O(1)
```
^vs-no-len

### Больше аллокаций
sync.Map внутри создаёт entry объекты для каждого ключа.
Это дополнительная нагрузка на GC. ^vs-allocations

---

## Простое правило выбора
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  По умолчанию: map + RWMutex (или просто Mutex)             │
│                                                             │
│  sync.Map только если ВСЕ условия:                          │
│    ✓ Ключи пишутся редко, читаются часто                    │
│    ✓ ИЛИ горутины работают с разными ключами                │
│    ✓ НЕ нужна итерация (Range)                              │
│    ✓ НЕ нужен len()                                         │
│    ✓ Много горутин (50+)                                    │
│    ✓ Замерил и убедился что быстрее                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
^vs-decision-rule

---

## Типичные use cases

| Сценарий | Лучший выбор | Почему |
|:---------|:-------------|:-------|
| Кэш конфигов | sync.Map | write-once, read-many |
| Кэш сессий | sync.Map | каждая горутина — свой user |
| Счётчики | map + Mutex | частые инкременты |
| Rate limiter | map + RWMutex | частые записи timestamps |
| Нужен len() | map + RWMutex | sync.Map не умеет |
| Нужен Range | map + RWMutex | Range у sync.Map медленный |
| Мало горутин (<10) | map + RWMutex | sync.Map overhead не окупится |
^vs-use-cases

---

## Цитата из документации Go

> "The Map type is optimized for two common use cases:
> (1) when the entry for a given key is only ever written once but read many times
> (2) when multiple goroutines read, write, and overwrite entries for disjoint sets of keys.
> In these two cases, use of a Map may significantly reduce lock contention compared to a Go map paired with a separate Mutex or RWMutex."

Обрати внимание: "two common use cases" — не "all use cases".
sync.Map — **специализированный** инструмент, не универсальная замена. ^vs-docs-quote

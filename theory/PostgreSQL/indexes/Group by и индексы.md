- B-tree хранит ключи **отсортированными** → PG может делать **GroupAggregate** без хеширования и без сортировки
- Без индекса: **HashAggregate** (хеш-таблица в памяти) или **Sort → GroupAggregate** (сортировка всех строк). С индексом: сразу GroupAggregate
- GROUP BY по выражению → нужен **expression индекс**: `GROUP BY LOWER(name)` → индекс на `LOWER(name)`
- **GIN/GiST** для GROUP BY **бесполезны** — не хранят данные отсортированными

---

## Без индекса — PG делает всё сам

```sql
SELECT status, COUNT(*) FROM orders GROUP BY status;
```

Два варианта:

```
-- Вариант 1: HashAggregate
-- Строит хеш-таблицу {status → count} в памяти
Seq Scan on orders → HashAggregate

-- Вариант 2: Sort + GroupAggregate
-- Сортирует ВСЕ строки по status, потом проходит последовательно
Seq Scan on orders → Sort → GroupAggregate
```

Оба читают **всю таблицу** + тратят CPU на хеширование или сортировку.

## С индексом — данные уже отсортированы

```sql
CREATE INDEX idx_status ON orders(status);

SELECT status, COUNT(*) FROM orders GROUP BY status;
```

B-tree по status хранит ключи так:

```
active → active → active → banned → banned → pending → pending → pending
```

PG идёт по индексу слева направо: группа "active" — 3 штуки, "banned" — 2, "pending" — 3. Не нужно ни сортировать, ни строить хеш-таблицу.

```
GroupAggregate
  → Index Only Scan using idx_status on orders   ← уже отсортировано
```

## Составной индекс и GROUP BY

```sql
CREATE INDEX idx_status_date ON orders(status, created_at);

-- ✅ Использует — GROUP BY по префиксу
SELECT status, COUNT(*) FROM orders GROUP BY status;

-- ✅ Использует — GROUP BY совпадает с индексом
SELECT status, DATE(created_at), COUNT(*)
FROM orders
GROUP BY status, created_at;

-- ❌ Не использует — пропущен первый столбец
SELECT DATE(created_at), COUNT(*)
FROM orders
GROUP BY created_at;
```

Правило левого префикса работает и для GROUP BY — ровно как для WHERE.

## GROUP BY + WHERE

```sql
CREATE INDEX idx_status_date ON orders(status, created_at);

SELECT created_at, COUNT(*)
FROM orders
WHERE status = 'pending'
GROUP BY created_at;
```

PG использует индекс: фильтрует по `status = 'pending'` (первый столбец), затем GroupAggregate по `created_at` (второй столбец, уже отсортирован внутри группы).

## GROUP BY по выражению

```sql
-- ❌ Индекс на name не поможет
SELECT LOWER(name), COUNT(*) FROM users GROUP BY LOWER(name);

-- ✅ Expression индекс
CREATE INDEX idx_lower_name ON users(LOWER(name));
SELECT LOWER(name), COUNT(*) FROM users GROUP BY LOWER(name);
```

## Когда индекс НЕ помогает для GROUP BY

- **Низкая селективность** — мало уникальных значений (3 статуса на 100M строк), планировщик выберет HashAggregate как дешевле
- **Маленькая таблица** — seq scan + HashAggregate быстрее чем обход индекса
- **GIN/GiST** — не хранят данные отсортированными
- **Hash индекс** — нет порядка, для GROUP BY бесполезен

## HashAggregate vs GroupAggregate

||HashAggregate|GroupAggregate|
|:--|:--|:--|
|Требует|Память (work_mem)|Отсортированные данные|
|Без индекса|Быстрее на малом кол-ве групп|Нужна сортировка|
|С индексом|Не нужен|Используется|
|Память|O(кол-во групп)|O(1)|
|Много групп|Может уйти на диск|Стабильно|

`work_mem` маленький + много групп → HashAggregate уходит на диск → **GroupAggregate по индексу быстрее**.

## Связь

- [[theory/PostgreSQL/indexes/B-tree индекс]] — отсортированные leaf pages = основа GroupAggregate
- [[Составные индексы (multi-column)]] — правило левого префикса для GROUP BY
- [[Expression индексы (functional)]] — GROUP BY LOWER(x) → expression индекс
- [[Селективность и издержки индексов]] — низкая селективность → HashAggregate выгоднее
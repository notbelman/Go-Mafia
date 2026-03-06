**Да** — B-tree хранит ключи отсортированными, и это ключевой момент.

### Без индекса — PostgreSQL делает всё сам
```
SELECT status, COUNT(*) FROM orders GROUP BY status;

-- Вариант 1: HashAggregate
-- Строит хеш-таблицу {status → count} в памяти, проходит все строки
Seq Scan on orders → HashAggregate

-- Вариант 2: Sort + GroupAggregate  
-- Сортирует все строки по status, потом проходит последовательно
Seq Scan on orders → Sort → GroupAggregate
```

Оба варианта читают всю таблицу + тратят CPU на хеширование или сортировку.

### С индексом — данные уже отсортированы
```sql
CREATE INDEX idx_status ON orders(status);

SELECT status, COUNT(*) FROM orders GROUP BY status;
```

B-tree по status хранит ключи так:
```
active → active → active → banned → banned → pending → pending → pending
```

PostgreSQL просто идёт по индексу слева направо: группа "active" — 3 штуки, группа "banned" — 2, группа "pending" — 3. Не нужно ни сортировать, ни строить хеш-таблицу.
```
-- В EXPLAIN:
GroupAggregate
  → Index Only Scan using idx_status on orders   ← уже отсортировано
```

### Когда НЕ помогает

- **Низкая селективность** — мало уникальных значений, планировщик решит что seq scan + HashAggregate дешевле
- **GROUP BY по выражению** — `GROUP BY LOWER(name)` не использует индекс на `name`, нужен индекс на `LOWER(name)`
- **Маленькая таблица** — seq scan дешевле
- **GIN/GiST индексы** — не хранят данные отсортированными, для GROUP BY бесполезны
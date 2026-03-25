- **Индексы на ключах JOIN** — главная оптимизация. PG **не создаёт индексы на FK** автоматически (MySQL создаёт)
- **Anti-join** (строки БЕЗ пары): `NOT EXISTS` > `LEFT JOIN WHERE IS NULL` > `NOT IN` (NULL-ловушка!)
- **Semi-join** (есть ли пара): `EXISTS` ≈ `IN` (PG оптимизирует оба). `JOIN` хуже — даёт дубликаты
- 10+ таблиц → планировщик перебирает 3.6M вариантов порядка. `join_collapse_limit = 8` (дефолт) — выше не перебирает

---

## Anti-join: найти строки БЕЗ пары

```sql
-- 1. NOT EXISTS — рекомендуется ✅
SELECT * FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- 2. LEFT JOIN WHERE IS NULL — тоже хорошо ✅
SELECT u.* FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;

-- 3. NOT IN — ОПАСНО ⚠️
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM orders);
-- Если в orders.user_id есть NULL → вернёт 0 строк (NULL-ловушка)
```

**Почему NOT IN опасен**: `id NOT IN (1, 2, NULL)` → для любого id: `id != NULL` → NULL → весь NOT IN = NULL → ни одна строка не пройдёт.

**Правило**: используй `NOT EXISTS`. Всегда корректно, PG оптимизирует.

## Semi-join: есть ли хотя бы одна пара

```sql
-- EXISTS — останавливается на первом найденном ✅
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- IN — PG оптимизирует аналогично EXISTS ✅
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);

-- JOIN — хуже: если у юзера 5 заказов → 5 строк (дубликаты) ❌
SELECT u.* FROM users u JOIN orders o ON u.id = o.user_id;
-- нужен DISTINCT → лишняя работа
```

## Self JOIN

```sql
-- Найти юзеров с одинаковым email (дубликаты)
SELECT a.id, b.id, a.email
FROM users a
JOIN users b ON a.email = b.email AND a.id < b.id;

-- Иерархия: сотрудник → руководитель
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

## Оптимизация

**Индексы на FK**: PG не создаёт автоматически. Без индекса → Nested Loop с Seq Scan внутри.
```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

**Устаревшая статистика**: `ANALYZE table` после массовых изменений.

**Много таблиц**: 3 таблицы = 6 вариантов порядка, 10 = 3.6 млн. `join_collapse_limit` (дефолт 8) — выше этого планировщик упрощает перебор.

**CTE**:
- PG < 12: **всегда** материализуется (барьер для оптимизатора)
- PG 12+: инлайнится по умолчанию. `MATERIALIZED` — принудительная материализация

## Алгоритмы JOIN (подробности в EXPLAIN)

| Алгоритм        | Когда                                       | Сложность                               |
| :-------------- | :------------------------------------------ | :-------------------------------------- |
| **Nested Loop** | Малая правая таблица, есть индекс           | O(n × m) worst, O(n × log m) с индексом |
| **Hash Join**   | Нет индекса, правая помещается в memory     | O(n + m)                                |
| **Merge Join**  | Обе таблицы отсортированы (или есть индекс) | O(n log n + m log m)                    |

PG выбирает автоматически по статистике. `SET enable_hashjoin = off` — для дебага, не для прода.

## Связь
- [[Типы JOIN]] — что делает каждый тип
- [[JOIN подводные камни]] — ON vs WHERE, NULL
- [[LATERAL JOIN]] — всегда Nested Loop
- [[Селективность и издержки индексов]] — индекс на FK = главная оптимизация JOIN

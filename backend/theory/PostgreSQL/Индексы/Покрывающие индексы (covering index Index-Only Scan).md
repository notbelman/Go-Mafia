- **Покрывающий индекс** — содержит все колонки запроса. PG читает **только индекс**, не ходит в таблицу → **Index-Only Scan** вместо Index Scan + Heap Fetch ^cov-def
- `INCLUDE (col)` — колонка **хранится** в индексе, но **не участвует** в сортировке/фильтрации. Меньше размер чем составной ^cov-include
- Нужен регулярный VACUUM (visibility map), иначе PG всё равно пойдёт в heap ^cov-vacuum

---

**Что это:** Индекс содержит все колонки запроса. PostgreSQL читает только индекс, **не ходит в таблицу.**

```sql
CREATE INDEX idx_email ON users(email) INCLUDE (name);
```

**Зачем:**

- Без INCLUDE: Index Scan → Heap Fetch (2 чтения)
- С INCLUDE: Index-Only Scan (1 чтение)

**INCLUDE vs составной индекс:**

- Составной `(A, B)` - B участвует в сортировке, можно фильтровать по B
- INCLUDE `(A) INCLUDE (B)` - B только хранится, меньше размер

**Когда использовать:**

- Частые SELECT с фиксированным набором колонок
- Аналитика/отчёты
- Hot paths в API

**Ограничения:**

- Visibility map - нужен регулярный VACUUM
- Размер индекса растёт

---

**Примеры с кодом:**

**Проблема без покрытия:**

```sql
CREATE INDEX idx_email ON users(email);
SELECT email, name FROM users WHERE email = 'user@example.com';

-- 1. Index Scan: ищем email → получаем TID
-- 2. Heap Fetch: идём в таблицу за name
```

**Решение с INCLUDE:**

```sql
CREATE INDEX idx_email ON users(email) INCLUDE (name);
SELECT email, name FROM users WHERE email = 'user@example.com';

-- Index-Only Scan: всё есть в индексе
```

**INCLUDE vs составной:**

```sql
-- Составной: name в сортировке
CREATE INDEX idx_composite ON users(email, name);

-- INCLUDE: name просто хранится
CREATE INDEX idx_covering ON users(email) INCLUDE (name);

-- Разница: составной позволяет WHERE name = X, INCLUDE - нет
```

**Составной + INCLUDE:**

```sql
CREATE INDEX idx_orders ON orders(status, created_at) 
INCLUDE (total_amount, customer_id);

SELECT total_amount, customer_id 
FROM orders 
WHERE status = 'pending' AND created_at > '2024-01-01';
-- Index-Only Scan
```

**Проверка:**

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT email, name FROM users WHERE email = 'x';

-- Index-Only Scan ✅ - работает
-- Index Scan + Heap Fetches ❌ - нужен INCLUDE
```

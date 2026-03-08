- `SELECT *` в проде — **антипаттерн**. Убивает **Index Only Scan** (PG вынужден ходить в heap за всеми колонками), тянет лишние данные по сети, ломается при добавлении колонок
- Перечисляй колонки явно: `SELECT id, name, email` → PG может использовать **covering index**, меньше I/O, меньше трафика
- Исключение: `EXISTS (SELECT * ...)` — PG оптимизирует, `*` не читается. И ad-hoc запросы в psql для дебага

---

## Почему плохо

### 1. Убивает Index Only Scan

```sql
CREATE INDEX ON users(email) INCLUDE (name);

SELECT * FROM users WHERE email = 'x@y.com';
-- Index Scan + Heap Fetch (нужен id, created_at, и т.д.)

SELECT email, name FROM users WHERE email = 'x@y.com';
-- Index Only Scan ✅ (всё есть в индексе)
```

`SELECT *` заставляет PG ходить в heap за **каждой** колонкой, даже если индекс покрывает нужные.

### 2. Лишние данные по сети

```
Таблица users: 20 колонок, средняя строка 500 байт
Нужно: id + name = 30 байт

SELECT *:           500 байт × 100K строк = 50 MB по сети
SELECT id, name:     30 байт × 100K строк =  3 MB по сети
```

### 3. Ломается при ALTER TABLE

```sql
-- Код: SELECT * FROM users → парсит результат по индексу колонки
-- ALTER TABLE users ADD COLUMN middle_name TEXT;
-- Теперь колонки сдвинулись → код может сломаться
```

Явные колонки = **контракт** между запросом и кодом. `SELECT *` = хрупкий контракт.

### 4. Тянет TOAST-поля

Большие поля (TEXT, JSONB > 2KB) хранятся в TOAST-таблице. `SELECT *` тянет их даже если не нужны = лишний I/O.

## Когда SELECT * допустим

- `EXISTS (SELECT * FROM ...)` — PG **не читает** данные, только проверяет наличие
- Ad-hoc запросы в psql / DBeaver для дебага
- CTE / подзапрос если сразу фильтруешь (PG инлайнит)

## Связь
- [[Покрывающие индексы (covering index Index-Only Scan)]] — SELECT * убивает Index Only Scan
- [[EXPLAIN красные флаги]] — Heap Fetches при Index Scan = возможно SELECT *
- [[Scan-ноды в EXPLAIN]] — Index Only Scan требует перечисления колонок

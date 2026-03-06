Все данные берём из индекса, в heap не ходим.
```
Index Only Scan using idx_email on users  (rows=1)
  Index Cond: (email = 'test@test.com')
  Heap Fetches: 0   ← идеально (может быть больше, тогда `VACUUM table` (обновит visibility map))
```

## Два условия

1. **Covering Index** — все SELECT-колонки в индексе
2. **Visibility Map актуальна** — PG знает что страницы не менялись

```sql
-- Обычный индекс
CREATE INDEX ON users(email);
SELECT email FROM users WHERE email = 'x';  -- Index Only Scan ✅

-- Составной индекс
CREATE INDEX ON users(status, age);
SELECT status, age FROM users WHERE status = 'active';  -- Index Only Scan ✅

-- INCLUDE (PG 11+)
CREATE INDEX ON users(status) INCLUDE (age);
SELECT status, age FROM users WHERE status = 'active';  -- Index Only Scan ✅
```

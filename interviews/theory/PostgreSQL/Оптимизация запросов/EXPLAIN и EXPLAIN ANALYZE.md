EXPLAIN — план выполнения (что PG собирается делать)
EXPLAIN ANALYZE — выполняет запрос + реальные метрики

## Синтаксис
```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE age > 25;
```

⚠️ ANALYZE выполняет запрос! Для INSERT/UPDATE/DELETE:
```sql
BEGIN;
EXPLAIN ANALYZE DELETE FROM users WHERE id = 1;
ROLLBACK;
```
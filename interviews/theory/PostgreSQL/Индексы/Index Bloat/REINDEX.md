Пересоздаёт индекс, **возвращает место на диск**.
### Когда нужен

- Index Bloat после массовых DELETE/UPDATE
- Индекс повреждён
- Индекс >> 40% от размера таблицы
### Варианты

```sql
-- Блокирует таблицу
REINDEX INDEX idx_users_email;

-- Без блокировки (PG 12+)
REINDEX INDEX CONCURRENTLY idx_users_email;

-- Вручную (если CONCURRENTLY недоступен)
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
DROP INDEX idx_users_email;
ALTER INDEX idx_users_email_new RENAME TO idx_users_email;
```
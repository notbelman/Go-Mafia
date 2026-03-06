**Уникальные индексы**

UNIQUE constraint автоматически создаёт B-tree индекс. Можно создать вручную:

```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```

**Когда создавать вручную вместо constraint:**

- Нужен `CONCURRENTLY` (без блокировки)
- Нужен частичный (`WHERE`)
- Нужен покрывающий (`INCLUDE`)

```sql
-- Частичный уникальный - только для активных
CREATE UNIQUE INDEX idx_email ON users(email) WHERE deleted_at IS NULL;
```

Подробнее про constraints -> [[Общая табличка]]
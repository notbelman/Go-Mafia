- **UNIQUE constraint** автоматически создаёт B-tree индекс. Можно создать уникальный индекс вручную — больше контроля ^ui-def
- Вручную когда нужен: `CONCURRENTLY` (без блокировки), `WHERE` (частичный), `INCLUDE` (покрывающий) ^ui-when-manual
- Частичный уникальный — мощная комбинация: `UNIQUE ... WHERE deleted_at IS NULL` (уникальность только среди активных) ^ui-partial

---

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

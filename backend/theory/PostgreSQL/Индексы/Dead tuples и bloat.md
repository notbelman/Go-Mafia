- **Dead tuples(MVCC)**: DELETE помечает строку мёртвой, UPDATE = DELETE + INSERT новой. В heap и индексе копится **мусор** → нужен VACUUM
- **Index bloat**: после массовых DELETE/UPDATE индекс занимает больше чем нужно. VACUUM чистит внутри, но **файл не уменьшает** → нужен REINDEX
- `CREATE INDEX CONCURRENTLY` — без блокировки таблицы (два прохода, **~2x дольше**). При ошибке индекс = INVALID
- `REINDEX CONCURRENTLY` (PG 12+) — пересоздание без блокировки

---

## Dead tuples (MVCC)

PostgreSQL не изменяет и не удаляет данные физически:

**DELETE:** строка помечается мёртвой. В индексе указатель остаётся.

**UPDATE:** старая версия помечается мёртвой + создаётся новая с новым TID + в индекс добавляется новый указатель.

```
До UPDATE:
  Индекс: alice@gmail.com → (0,2)
  Heap:   (0,2) = {id:3, email:'alice@gmail.com'}

После UPDATE email → 'alice_new@gmail.com':
  Индекс: alice@gmail.com     → (0,2)  ← мёртвый указатель
          alice_new@gmail.com → (0,4)  ← новый
  Heap:   (0,2) = мёртвая строка
          (0,4) = {id:3, email:'alice_new@gmail.com'}
```

## Index Bloat

VACUUM чистит и heap, и индексы. Но **физический размер файлов не уменьшает** — место внутри становится свободным для переиспользования.

```
После 1M DELETE + VACUUM:
  Индекс: внутри свободно, файл всё ещё 2GB
  Новые INSERT переиспользуют место
  Нет INSERT → файл так и останется 2GB
```

### Как заметить

```sql
SELECT
  pg_size_pretty(pg_relation_size('users')) AS table_size,
  pg_size_pretty(pg_relation_size('idx_users_email')) AS index_size;

-- Индекс >> 40% от таблицы → вероятно bloat
```

## REINDEX — пересоздание индекса

```sql
-- Блокирует таблицу на запись
REINDEX INDEX idx_users_email;

-- Без блокировки (PG 12+)
REINDEX INDEX CONCURRENTLY idx_users_email;

-- Вручную (если CONCURRENTLY недоступен)
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
DROP INDEX idx_users_email;
ALTER INDEX idx_users_email_new RENAME TO idx_users_email;
```

## CREATE INDEX CONCURRENTLY

Обычный `CREATE INDEX` блокирует INSERT/UPDATE/DELETE на всё время создания. На 100M строк — минуты простоя.

```sql
CREATE INDEX CONCURRENTLY idx_email ON users(email);
```

**Два прохода:** сканирует таблицу → ждёт завершения текущих транзакций → второй проход добавляет изменения. Работает **~2x дольше**.

**Если упал** (дубликат в UNIQUE):

```sql
-- Индекс остаётся INVALID (не используется, но занимает место)
SELECT indexrelid::regclass, indisvalid
FROM pg_index WHERE NOT indisvalid;

-- Удалить и пересоздать
DROP INDEX CONCURRENTLY idx_email;
CREATE INDEX CONCURRENTLY idx_email ON users(email);
```

## Связь
- [[Heap страницы и TID]] — dead tuples в heap
- [[MVCC (Multi-Version Concurrency Control)]] — почему PG не удаляет физически
- [[B-tree индекс]] — fillfactor снижает page splits при UPDATE
- [[HOT updates]] — UPDATE без обновления индекса
- [[Мониторинг индексов]] — как отслеживать bloat

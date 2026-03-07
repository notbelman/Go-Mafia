- **HOT** (Heap-Only Tuple) — UPDATE без обновления индекса, если: изменяемая колонка **не в индексе** + новая версия на **той же странице**
- Экономит: не нужно добавлять запись в индекс + dead tuple в индексе. **В 2-3x быстрее** обычного UPDATE
- Для HOT нужен **fillfactor < 100** (место на странице для новой версии). UPDATE-heavy → `fillfactor = 70-80`

---

## Как работает

Обычный UPDATE:
```
1. Пометить старую строку мёртвой
2. Вставить новую строку (возможно на другой странице)
3. Добавить новую запись в КАЖДЫЙ индекс   ← дорого
```

HOT UPDATE:
```
1. Пометить старую строку мёртвой
2. Вставить новую строку НА ТОЙ ЖЕ странице
3. Старая строка → redirect pointer → новая строка
4. Индексы НЕ ТРОГАЕМ                              ← экономия
```

Индекс по-прежнему ссылается на старый TID. При чтении PG идёт по redirect chain: старый TID → новая строка.

## Условия для HOT

Оба условия одновременно:
1. **Изменяемая колонка не входит ни в один индекс** — иначе индекс нужно обновить
2. **На странице есть место** для новой версии строки — иначе новая строка уйдёт на другую страницу

```sql
-- Индекс на email
CREATE INDEX idx_email ON users(email);

UPDATE users SET name = 'Alice' WHERE id = 1;
-- name не в индексе → HOT ✅

UPDATE users SET email = 'new@mail.ru' WHERE id = 1;
-- email в индексе → обычный UPDATE ❌
```

## Fillfactor

```sql
-- Дефолт 100% — вся страница заполняется → мало места для HOT
CREATE TABLE users (...) WITH (fillfactor = 70);

-- Или для индекса
CREATE INDEX idx ON users(email) WITH (fillfactor = 70);
```

70% → 30% страницы свободно для HOT updates. Таблица больше, но UPDATE быстрее.

## Мониторинг

```sql
SELECT
  n_tup_upd AS total_updates,
  n_tup_hot_upd AS hot_updates,
  ROUND(100.0 * n_tup_hot_upd / NULLIF(n_tup_upd, 0), 1) AS hot_pct
FROM pg_stat_user_tables
WHERE relname = 'users';

-- hot_pct > 90% = хорошо
-- hot_pct < 50% = возможно нужен fillfactor или пересмотр индексов
```

## Связь
- [[theory/PostgreSQL/indexes/B-tree индекс]] — fillfactor на индексе тоже влияет на page splits
- [[Dead tuples и bloat]] — HOT уменьшает мусор в индексах
- [[Мониторинг индексов]] — n_tup_hot_upd в pg_stat_user_tables

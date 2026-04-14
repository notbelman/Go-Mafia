- **GIN** (Generalized Inverted Index) — инвертированный индекс: `value → [row1, row2, ...]`. Для **массивов, JSONB, full-text search**
- Внутри: **B-tree ключей** + **posting list** (список TID) для каждого ключа. Большой posting list → posting tree
- Быстрый SELECT, **медленный INSERT** (fastupdate буферизует). Размер **50-200%** от таблицы
- **jsonb_ops** (дефолт) — все операторы. **jsonb_path_ops** — только `@>`, но **меньше и быстрее**

---

## Внутренняя структура

Ключи хранятся в B-tree для быстрого поиска. К каждому ключу — posting list (список TID).

> **GIN = B-tree ключей + posting list/tree для каждого ключа**

```
posts: id=1 tags=['rust', 'backend']
       id=2 tags=['rust', 'systems']

GIN B-tree ключей:
        [rust]
       /      \
 [backend]   [systems]

Posting lists:
'backend' → [1]
'rust'    → [1, 2]       ← если список большой → posting tree
'systems' → [2]
```

## Создание

```sql
CREATE INDEX idx_tags ON posts USING GIN(tags);
CREATE INDEX idx_data ON events USING GIN(data jsonb_path_ops);
CREATE INDEX idx_search ON articles USING GIN(to_tsvector('russian', content));
```

## Полезные операторы

| Оператор | Описание | Пример |
|---|---|---|
| `@>` | Содержит | `tags @> '{rust}'` |
| `<@` | Содержится в | `tags <@ '{rust, go}'` |
| `&&` | Пересекается | `tags && '{rust, go}'` |
| `?` | Ключ существует (JSONB) | `data ? 'name'` |
| `@@` | Full-text match | `tsv @@ to_tsquery('rust')` |

## jsonb_ops vs jsonb_path_ops

```sql
-- Дефолт: все операторы, больше размер
CREATE INDEX idx ON events USING GIN(data);

-- Только @>, меньше и быстрее
CREATE INDEX idx ON events USING GIN(data jsonb_path_ops);
```

## fastupdate

`fastupdate = on` (по умолчанию) — буферизует вставки в pending list, ускоряя INSERT. При заполнении pending list сливается в основной индекс. `gin_pending_list_limit` — размер до принудительного слияния.

## Когда использовать / не использовать

**Да:** массивы, JSONB, full-text, частые SELECT, редкие INSERT.

**Нет:** простые скалярные значения (B-tree лучше), частые вставки, маленькие таблицы.

## Связь
- [[B-tree индекс]] — для скаляров лучше B-tree
- [[GiST индекс (Generalized Search Tree)]] — альтернатива: быстрее пишет, медленнее ищет
- [[pg_trgm и поиск подстроки]] — GIN + pg_trgm для LIKE '%text%'

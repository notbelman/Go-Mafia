**Что это:** Инвертированный индекс — вместо `row → value` хранит `value → [row1, row2, ...]`

## Внутренняя структура

Ключи (отдельные элементы) хранятся в **B-tree дереве** для быстрого поиска.
К каждому ключу привязан **posting list** — список TID (указателей на строки).
Если posting list большой, он превращается в **posting tree** (тоже B-tree).

> **GIN = B-tree ключей + posting list/tree для каждого ключа**

## Пример
```
posts: id=1 tags=['rust', 'backend']
       id=2 tags=['rust', 'systems']

GIN B-tree ключей:
        [rust]
       /      \
 [backend]   [systems]   ← каждый ключ → posting list

Posting lists:
'backend' → [1]
'rust'    → [1, 2]       ← если список большой, становится posting tree
'systems' → [2]
```

## Использовать

- Массивы, JSONB, full-text search
- Поиск "содержит", "пересекается", "ключ существует"
- Частые SELECT, редкие INSERT/UPDATE

## Не использовать

- Простые скалярные значения (B-tree лучше)
- Частые вставки (медленный maintenance)
- Маленькие таблицы (overhead не окупится)

## Создание
```sql
CREATE INDEX idx_tags ON posts USING GIN(tags);
CREATE INDEX idx_data ON events USING GIN(data jsonb_path_ops);
CREATE INDEX idx_search ON articles USING GIN(to_tsvector('russian', content));
```

## Полезные операторы

| Оператор | Описание | Пример |
|----------|----------|--------|
| `@>` | Содержит | `tags @> '{rust}'` |
| `<@` | Содержится в | `tags <@ '{rust, go}'` |
| `&&` | Пересекается | `tags && '{rust, go}'` |
| `?` | Ключ существует (JSONB) | `data ? 'name'` |
| `@@` | Full-text match | `tsv @@ to_tsquery('rust')` |

## Важно

- Поиск по ключам внутри GIN идёт через B-tree → нахождение ключа за **O(log n)**
- `fastupdate = on` (по умолчанию) — буферизует вставки в pending list, ускоряя INSERT
- `gin_pending_list_limit` — размер pending list до принудительного слияния
- **B-tree** (Balanced tree) — дефолтный индекс PG. Сбалансированное дерево, оптимизированное для диска: узел = страница **8 КБ**, **100-300 потомков**, 1M строк = **3-4 уровня**
- **Поддерживает**: =, `<`, `>`, `BETWEEN`, `ORDER BY`, `IS NULL`, `LIKE 'prefix%'`. 
  **Не поддерживает**: `LIKE '%middle%'`, `!=`, `NOT IN`
- Leaf pages связаны **linked list** → range scan идёт по цепочке без возврата к root
- **Fillfactor** (дефолт 90%) — оставить место на странице для UPDATE без page split. Для UPDATE-heavy: `fillfactor=70`

---

## Структура

Дерево из страниц по 8 КБ. Три типа:

- **Metapage** — указатель на root
- **Internal pages** — ключ + указатель на дочернюю страницу индекса
- **Leaf pages** — ключ + TID (указатель на строку в heap)

```
         [Metapage → Root]
              │
     ┌────[30, 60]────┐
     │        │        │
  [10,20]  [40,50]  [70,80,90]    ← internal pages
   │ │ │    │ │ │    │ │ │ │
  [leafs linked together →→→]     ← leaf pages (связаны списком)
```

## Balanced tree vs Binary tree

| | Binary Tree | B-tree |
|---|---|---|
| Потомков | 2 | 100-300 |
| Балансировка | Может быть нет | Всегда |
| 1M строк | ~20 уровней | 3-4 уровня |
| Оптимизирован для | RAM | Диск (page-oriented) |

"B" = **Balanced**, не "Binary".

## Поддерживаемые операции

```sql
WHERE id = 123              -- ✅ точечный поиск
WHERE created_at > '2024'   -- ✅ range
ORDER BY name               -- ✅ сортировка (и ASC и DESC)
WHERE email LIKE 'alex%'    -- ✅ префиксный поиск (с varchar_pattern_ops)
WHERE email LIKE '%alex%'   -- ❌ нужен GIN + pg_trgm
WHERE status != 'active'    -- ❌ не использует индекс
```

## Fillfactor

```sql
-- Дефолт: 90% заполнение страницы
CREATE INDEX idx ON users(email);

-- UPDATE-heavy таблица: 70% → место для HOT updates
CREATE INDEX idx ON users(email) WITH (fillfactor = 70);
```

Меньше fillfactor → больше места для обновлений без page split, но индекс больше.

## Deduplication (PG 13+)

Одинаковые ключи хранятся компактнее: вместо `[key→TID1], [key→TID2]` — `[key→[TID1, TID2]]`. Экономия 30-50% на колонках с повторами (status, boolean).

## Ограничения

- Максимум **~2712 байт на ключ** (1/3 страницы)
- Неэффективен при низкой селективности (пол, булевы) — планировщик выберет seq scan

## Связь
- [[Heap страницы и TID]] — leaf pages хранят TID → ссылку в heap
- [[B-Tree vs LSM-Tree]] — сравнение подходов к хранению
- [[Составные индексы (multi-column)]] — B-tree на несколько колонок
- [[HOT updates]] — fillfactor влияет на возможность HOT

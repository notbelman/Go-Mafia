`work_mem` — это лимит памяти на одну операцию сортировки или хэширования.

Не на запрос, не на соединение — на **одну операцию**. Один запрос может иметь несколько Sort или Hash узлов, каждый получит свой work_mem.

```sql
SHOW work_mem;  -- по умолчанию 4MB
SET work_mem = '256MB';
```

## Что использует work_mem

- **Hash Join** — хэш-таблица
- **Hash Aggregate** — GROUP BY через хэш
- **Sort** — сортировка в памяти
- **Bitmap Heap Scan** — битовая карта

## Если не влезло

Операция уходит на диск (temporary files) — резко медленнее.

В EXPLAIN ANALYZE видно:

```
Sort Method: external merge  Disk: 102400kB   ← не влезло
Sort Method: quicksort  Memory: 25kB          ← влезло
```
- Чек-лист **красных флагов** в EXPLAIN ANALYZE: что искать, почему плохо, как чинить
- Главные: **Seq Scan + Filter с большим Rows Removed**, **actual rows >> estimated rows**, **Nested Loop без индекса**, **external sort / Batches на диске**
- Правило: сначала `EXPLAIN (ANALYZE, BUFFERS)`, потом ищи флаги сверху вниз по этому списку

---

## 1. Seq Scan + Filter на большой таблице

```
Seq Scan on orders  (actual rows=50)
  Filter: (status = 'pending')
  Rows Removed by Filter: 999950     ← прочитали 1M, оставили 50
```

**Проблема**: читаем всю таблицу, выкидываем 99.99%.

**Чинить**: `CREATE INDEX ON orders(status)` или частичный индекс `WHERE status = 'pending'`.

## 2. actual rows >> estimated rows

```
Index Scan on orders  (cost=... rows=10) (actual rows=50000)
                                    │                  │
                              оценка: 10        реально: 50000
```

**Проблема**: планировщик ошибся в 5000 раз → выбрал неоптимальный план.

**Чинить**: `ANALYZE orders` (обновить статистику). Если не помогло → `ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000` (больше сэмплов).

## 3. Nested Loop без индекса на внутренней

```
Nested Loop  (actual time=0.5..12000.0 rows=1000)
  ->  Seq Scan on users (actual rows=1000 loops=1)
  ->  Seq Scan on orders (actual rows=500 loops=1000)  ← 1000 × полный скан!
```

**Проблема**: 1000 × Seq Scan = O(N × M).

**Чинить**: `CREATE INDEX ON orders(user_id)`. Nested Loop + Index Scan = O(N × log M).

## 4. Index Scan + Filter с большим Rows Removed

```
Index Scan using idx_status on users  (actual rows=80)
  Index Cond: (status = 'active')
  Filter: (age > 25)
  Rows Removed by Filter: 420    ← 500 из индекса, 420 выкинули
```

**Проблема**: 420 лишних random reads в heap.

**Чинить**: составной индекс `CREATE INDEX ON users(status, age)` — Filter исчезнет.

## 5. External Sort / Hash Batches на диске

```
Sort Method: external merge  Disk: 98304kB    ← сортировка на диске
Hash Batches: 8  Disk Usage: 32768kB           ← хэш на диске
```

**Проблема**: work_mem не хватает → уход на диск → в разы медленнее.

**Чинить**: `SET work_mem = '256MB'` (для сессии) или индекс по sort key.

## 6. Lossy bitmap

```
Bitmap Heap Scan
  Heap Blocks: exact=0 lossy=1200    ← bitmap не влез
```

**Проблема**: bitmap схлопнулся до страниц → Recheck проверяет все строки на странице.

**Чинить**: `SET work_mem = '256MB'`.

## 7. Index Only Scan + Heap Fetches > 0

```
Index Only Scan using idx_email on users
  Heap Fetches: 5000    ← ходим в heap (visibility map устарела)
```

**Проблема**: должен читать только индекс, но visibility map не актуальна.

**Чинить**: `VACUUM users`.

## 8. Workers Launched < Workers Planned

```
Gather
  Workers Planned: 4
  Workers Launched: 1    ← запустился только 1 из 4
```

**Проблема**: не хватило параллельных воркеров.

**Чинить**: `max_parallel_workers`, `max_parallel_workers_per_gather`.

## 9. CTE Scan (PG < 12 или MATERIALIZED)

```
CTE Scan on heavy_cte  (actual rows=1000000)
  ->  Seq Scan on huge_table ...    ← материализовался весь
```

**Проблема**: CTE материализовался целиком, хотя нужна часть.

**Чинить**: PG 12+ — убрать `MATERIALIZED`. Или переписать CTE в подзапрос.

## 10. OFFSET на большом значении

```
Limit  (actual rows=10)
  ->  Index Scan (actual rows=100010)    ← прочитал 100K чтобы пропустить
```

**Проблема**: `OFFSET 100000 LIMIT 10` → читает 100010 строк.

**Чинить**: cursor-based pagination: `WHERE id > last_seen_id LIMIT 10`.

## Быстрый чек-лист

```
□ Rows Removed by Filter > 90% прочитанных?     → индекс
□ actual rows >> estimated rows?                  → ANALYZE
□ Nested Loop + Seq Scan на внутренней?          → индекс на JOIN key
□ Sort Method: external merge?                    → work_mem или индекс
□ Hash Batches > 1?                               → work_mem
□ Heap Blocks: lossy > 0?                         → work_mem
□ Heap Fetches > 0 при Index Only Scan?           → VACUUM
□ Workers Launched < Planned?                      → max_parallel_workers
```

## Связь
- [[EXPLAIN основы]] — как читать план
- [[Scan-ноды в EXPLAIN]] — Seq/Index/Index Only Scan
- [[Bitmap Scan]] — exact vs lossy
- [[JOIN-ноды в EXPLAIN]] — Nested Loop/Hash/Merge
- [[Вспомогательные ноды EXPLAIN]] — Sort, Aggregate, Gather

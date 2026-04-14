- **Sort** — сортировка для ORDER BY / Merge Join / GroupAggregate. Влезло в work_mem → **quicksort**. Не влезло → **external merge sort** (на диске, медленно)
- **HashAggregate** — GROUP BY через хэш-таблицу. O(кол-во групп) памяти. Много групп → может уйти на диск
- **GroupAggregate** — GROUP BY по уже отсортированным данным (индекс). O(1) памяти, **стабильно**
- **Gather / Parallel** — параллельный план: `Workers Planned: 2`. Реальное ускорение ≈ 2-3x, не линейное
- **Append** — UNION ALL и партиционированные таблицы. **MergeAppend** — UNION ALL + ORDER BY

---

## Sort

```
Sort  (actual time=12.5..15.2 rows=50000)
  Sort Key: created_at
  Sort Method: quicksort  Memory: 4096kB    ← влезло в work_mem ✅
```

```
Sort  (actual time=120.5..180.2 rows=5000000)
  Sort Key: created_at
  Sort Method: external merge  Disk: 98304kB  ← не влезло, на диске ❌
```

**External merge sort** — в разы медленнее. Решения:
- `SET work_mem = '256MB'` — больше памяти для сортировки
- Индекс по sort key — Sort исчезнет из плана

## HashAggregate vs GroupAggregate

```sql
SELECT status, COUNT(*) FROM orders GROUP BY status;
```

**HashAggregate** — строит хэш-таблицу `{status → count}`:
```
HashAggregate  (rows=5)
  Group Key: status
  Batches: 1  Memory Usage: 24kB          ← мало групп, влезло ✅
  ->  Seq Scan on orders
```

```
HashAggregate  (rows=500000)
  Batches: 16  Disk Usage: 32768kB         ← много групп, на диске ❌
```

**GroupAggregate** — данные уже отсортированы (индекс), идёт последовательно:
```
GroupAggregate
  Group Key: status
  ->  Index Only Scan using idx_status    ← уже отсортировано, O(1) памяти
```

| | HashAggregate | GroupAggregate |
|:--|:--|:--|
| Требует | Память (work_mem) | Отсортированные данные |
| Память | O(кол-во групп) | O(1) |
| Много групп | Может → диск | Стабильно |
| С индексом | Не нужен | Использует |

## Gather / Parallel

```
Gather  (actual rows=100000)
  Workers Planned: 2
  Workers Launched: 2                      ← реально запустились
  ->  Parallel Seq Scan on orders          ← каждый воркер сканирует часть
        Filter: (amount > 1000)
```

PG делит таблицу между воркерами. Gather собирает результаты.

**Parallel Index Scan**, **Parallel Hash Join**, **Parallel Bitmap Heap Scan** — тоже бывают.

`Workers Launched < Workers Planned` → не хватило `max_parallel_workers`.

Ускорение **не линейное**: Gather + merge результатов = overhead. 2 воркера ≈ 1.5-2x ускорение.

## CTE Scan / Subquery Scan

```
CTE Scan on cte_name  (actual rows=100)
  ->  ... (материализованный CTE)
```

**PG < 12**: CTE **всегда** материализуется (барьер для оптимизатора).
**PG 12+**: инлайнится по умолчанию. `WITH cte AS MATERIALIZED (...)` — принудительная материализация.

**Subquery Scan** — обёртка над подзапросом. Обычно "бесплатная" (passthrough).

## Append / MergeAppend

```sql
SELECT * FROM orders_2023
UNION ALL
SELECT * FROM orders_2024;
```

```
Append
  ->  Seq Scan on orders_2023
  ->  Seq Scan on orders_2024
```

**Партиционированные таблицы** — PG автоматически делает Append, пропуская ненужные партиции (partition pruning):

```
Append
  ->  Seq Scan on orders_2024_q1    ← только нужная партиция
  Subplans Removed: 3               ← 3 партиции пропущены
```

**MergeAppend** — UNION ALL с ORDER BY: сливает отсортированные потоки.

## Limit / Offset

```
Limit  (actual rows=10)
  ->  Index Scan using idx_created on orders  (actual rows=10)
```

Limit останавливает нижний узел рано — Index Scan прочитал только 10 строк, не всю таблицу. **Но**: `OFFSET 100000 LIMIT 10` → прочитает 100010 строк → cursor-based pagination лучше.

## Связь
- [[EXPLAIN основы]] — как читать Sort Method, Batches, Workers
- [[Group by и индексы]] — когда GroupAggregate vs HashAggregate
- [[EXPLAIN красные флаги]] — external sort, Batches > 1, Workers < Planned

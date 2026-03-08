- `EXPLAIN` — план (что PG собирается делать). `EXPLAIN ANALYZE` — план + **реальные метрики** (выполняет запрос!)
- **Cost** = условные единицы, не секунды. `seq_page_cost=1.0`, `random_page_cost=4.0`, `cpu_tuple_cost=0.01`. Index Scan на много строк может быть **дороже** Seq Scan (random × 4)
		`cost = (pages_read × page_cost) + (rows × cpu_tuple_cost)`
- **actual time × loops** = реальное время. `actual time=0.05 loops=1000` → **50 мс**, не 0.05
- Читать план **снизу вверх, изнутри наружу**. Самый вложенный узел выполняется первым
- `EXPLAIN ANALYZE` на INSERT/UPDATE/DELETE — **оборачивай в транзакцию** с ROLLBACK

---

## Синтаксис

```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE age > 25;

-- Для мутирующих запросов:
BEGIN;
EXPLAIN ANALYZE DELETE FROM users WHERE id = 1;
ROLLBACK;
```

## Как читать output

```
Seq Scan on users  (cost=0.00..35.50 rows=500 width=64) (actual time=0.01..0.42 rows=487 loops=1)
                         │      │       │       │                 │      │        │       │
                   startup   total   оценка   байт/           старт   конец  реальные   кол-во
                   cost      cost    строк    строка            мс      мс    строки   выполнений
```

**cost**: `0.00` — startup (время до первой строки), `35.50` — total (все строки).

**rows**: `rows=500` — оценка планировщика. `rows=487` — реально. Большое расхождение = устаревшая статистика → `ANALYZE table`.

**loops**: узел выполнился N раз (внутри Nested Loop). **Реальное время = actual time × loops.**

```
Index Scan on orders (actual time=0.01..0.05 rows=3 loops=1000)
                                  0.05 × 1000 = 50 мс реально
```

## Cost формула

```
cost = (pages_read × page_cost) + (rows × cpu_tuple_cost)
```

| Параметр | Значение | Что |
|:--|:--|:--|
| `seq_page_cost` | 1.0 | Последовательное чтение страницы |
| `random_page_cost` | 4.0 | Случайное чтение (индекс) |
| `cpu_tuple_cost` | 0.01 | Обработка одной строки |

**SSD**: можно снизить `random_page_cost` до **1.1-1.5** (random ≈ sequential на SSD).

## BUFFERS

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...

Buffers: shared hit=128 read=32
                  │          │
            из кэша    с диска (плохо если много)
```

`shared hit` — страницы из shared_buffers (быстро). `read` — с диска (медленно).

## Связь
- [[Scan-ноды в EXPLAIN]] — Seq Scan, Index Scan, Index Only Scan
- [[EXPLAIN красные флаги]] — что искать в плане
- [[Селективность и издержки индексов]] — почему планировщик выбирает Seq Scan

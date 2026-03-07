- **Seq Scan** — читает ВСЕ страницы подряд**(sequence I/O)**. Выбирается когда нет индекса, нужно >5-10% таблицы, или таблица маленькая. Не всегда плохо
- **Index Scan** — ищет в индексе → достаёт строки из heap **random I/O**. Выгоден при <5-10% строк. Каждая строка = отдельный random read
- **Index Only Scan** — всё из индекса, **в heap не ходим**. Нужен covering index + актуальная visibility map (VACUUM). `Heap Fetches: 0` = идеал
- **Filter** = красный флаг: `Rows Removed by Filter: 99990` → прочитали 100K, оставили 10. Нужен индекс или **составной индекс** чтобы убрать Filter
- Порог переключения Index→Seq: **~5-10%** таблицы. На SSD порог выше (random дешевле)

---

## Seq Scan

```
Seq Scan on users  (cost=0.00..35.50 rows=2000 width=64)
```

`cost = pages × 1.0 + rows × 0.01`. Последовательное чтение — дешёвое.

**Seq Scan + Filter**:
```
Seq Scan on t1  (actual rows=10)
  Filter: (col = 'x')
  Rows Removed by Filter: 99990  ← прочитали 100К, оставили 10
```

`Rows Removed by Filter` большой → нужен индекс на `col`.

## Index Scan

```
Index Scan using users_pkey on users  (cost=0.29..8.30 rows=1)
  Index Cond: (id = 42)
```

Индекс → TID → **random read** в heap за каждой строкой.

```
cost = (index_pages × 4.0) + (rows × 0.01) + (rows × 4.0) + (rows × 0.01)
                random I/O                     heap random I/O
```

**Почему может быть дороже Seq Scan**: 30% таблицы через индекс = тысячи random reads × 4.0 vs один проход × 1.0.

**Index Scan + Filter**:
```
Index Scan using idx_status on users  (actual rows=80)
  Index Cond: (status = 'active')     ← 500 строк из индекса
  Filter: (age > 25)                  ← из 500 оставили 80
  Rows Removed by Filter: 420         ← 420 лишних походов в heap
```

Решение — составной индекс:
```sql
CREATE INDEX ON users(status, age);
-- Index Cond: ((status = 'active') AND (age > 25))
-- Filter исчез, 80 походов вместо 500
```

## Index Only Scan

```
Index Only Scan using idx_email on users  (rows=1)
  Index Cond: (email = 'test@test.com')
  Heap Fetches: 0   ← идеально
```

Два условия:
1. **Covering index** — все SELECT-колонки в индексе (или INCLUDE)
2. **Visibility map актуальна** — VACUUM обновляет

```sql
CREATE INDEX ON users(email) INCLUDE (name);
SELECT email, name FROM users WHERE email = 'x';  -- Index Only Scan ✅
```

`Heap Fetches > 0` → давно не было VACUUM → `VACUUM users`.

## Сводка: когда что

```
                    Seq Scan          Index Scan        Index Only Scan
Когда              >5-10% таблицы    <5-10% таблицы    covering index
I/O                sequential        random             random (только индекс)
Ходит в heap       всегда            да                 нет (если VM актуальна)
```

## Связь
- [[EXPLAIN основы]] — как читать cost и actual time
- [[Bitmap Scan]] — промежуточный вариант между Seq и Index Scan
- [[Составные индексы (multi-column)]] — убирает Filter из Index Scan
- [[Покрывающие индексы (covering index Index-Only Scan)]] — INCLUDE для Index Only Scan

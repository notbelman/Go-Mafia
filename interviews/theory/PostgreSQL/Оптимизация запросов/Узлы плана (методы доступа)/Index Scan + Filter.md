Часть условий отрабатывает в индексе, часть — после чтения из heap.
```
Index Scan using idx_status on users  (actual rows=80)
  Index Cond: (status = 'active')   ← нашли 500 строк в индексе
  Filter: (age > 25)                ← из 500 оставили 80
  Rows Removed by Filter: 420
```

## Проблема

Index Cond нашёл 500 TID → 500 походов в heap → Filter выкинул 420.

Лишние чтения из heap.

## Решение — составной индекс
```sql
CREATE INDEX ON users(status, age);
```
```
Index Scan using idx_status_age on users  (actual rows=80)
  Index Cond: ((status = 'active') AND (age > 25))
```

Filter исчез, 80 походов в heap вместо 500.

## Порядок колонок в составном индексе

Equality (=) первыми, range (>, <) последними.
```sql
-- WHERE status = 'active' AND age > 25
CREATE INDEX ON users(status, age);  ✅
CREATE INDEX ON users(age, status);  ❌
```
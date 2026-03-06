
Синтаксический сахар для ON по одноимённым колонкам.

## USING
```sql
-- Эквивалентны:
SELECT * FROM t1 JOIN t2 ON t1.id = t2.id;
SELECT * FROM t1 JOIN t2 USING (id);

-- Несколько колонок:
SELECT * FROM t1 JOIN t2 USING (id, region);
```

Отличие: USING убирает дубликат колонки из SELECT *.

## NATURAL

Автоматический USING по всем одноимённым колонкам.
```sql
SELECT * FROM t1 NATURAL JOIN t2;
```

⚠️ Опасно: добавил колонку с тем же именем — сломал запрос. Не использовать в проде.
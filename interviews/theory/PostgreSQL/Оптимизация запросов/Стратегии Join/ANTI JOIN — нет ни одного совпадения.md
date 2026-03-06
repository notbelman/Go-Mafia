Модификатор стратегий JOIN в EXPLAIN. Вернуть строку если **нет ни одного совпадения.**
```sql
SELECT * FROM t1 WHERE NOT EXISTS (SELECT 1 FROM t2 WHERE t2.id = t1.id);
```

**В EXPLAIN выглядит как** - Стратегия + модификатор:
```
Hash Anti Join
Nested Loop Anti Join
Merge Anti Join
```
## Альтернативы (хуже)
```sql
-- LEFT JOIN + IS NULL (работает, но читается хуже)
SELECT * FROM t1
LEFT JOIN t2 ON t2.id = t1.id
WHERE t2.id IS NULL;

-- NOT IN (опасно с NULL!)
SELECT * FROM t1 WHERE id NOT IN (SELECT id FROM t2);
-- Если в t2 есть id = NULL → вернёт 0 строк
```


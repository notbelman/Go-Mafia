Модификатор стратегий JOIN в EXPLAIN. Вернуть строку если **есть хотя бы одно совпадение** _(sql EXIST)_. Без дубликатов.
```sql
SELECT * FROM t1 WHERE EXISTS (SELECT 1 FROM t2 WHERE t2.id = t1.id);
```

**В EXPLAIN выглядит как** - Стратегия + модификатор:
```
Hash Semi Join
Nested Loop Semi Join
Merge Semi Join
```
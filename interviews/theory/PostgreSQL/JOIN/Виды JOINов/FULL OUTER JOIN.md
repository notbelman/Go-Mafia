
LEFT JOIN + RIGHT JOIN вместе

Все строки из обеих таблиц, NULL где нет пары с любой стороны
```sql
SELECT * FROM t1
FULL OUTER JOIN t2 ON t1.id = t2.id;
```

**Пример:**
t1: (1), (1), (3)
t2: (1), (1), (4), (5)

**Результат:**

| t1.id | t2.id |
| ----- | ----- |
| 1     | 1     |
| 1     | 1     |
| 1     | 1     |
| 1     | 1     |
| 3     | NULL  |
| NULL  | 4     |
| NULL  | 5     |
	
⚠️ В MySQL нет FULL JOIN - нужно эмулировать через UNION
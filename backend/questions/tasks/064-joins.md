---
type: task
companies:
  - GroupIB
topic: PostgreSQL
subtopic:
  - JOIN
  - LEFT JOIN
  - INNER JOIN
title: Результат выполнения запросов с разными типами JOIN
---

## Условие

Есть две таблицы A и B. Какой будет результат выполнения каждого запроса?

## Пример
```
A:
| x |
|----|
| 1  |
| 2  |

B:
| y |
|----|
| 1  |
| 1  |
```
```sql
SELECT A.x, B.y FROM A LEFT JOIN B ON B.y = A.x;
SELECT A.x, B.y FROM A LEFT JOIN B ON A.x = B.y;
SELECT A.x, B.y FROM A INNER JOIN B ON B.y = A.x;
```

## Решение
```sql
```
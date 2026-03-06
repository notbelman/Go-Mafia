---
type: task
companies:
  - Wildberries
topic: PostgreSQL
subtopic:
  - Composite Index
  - Query Optimization
  - Index Column Order
title: Оптимизация запроса по таблице name/sex/year — порядок полей в индексе
---

## Условие

Как максимально разогнать этот запрос? Какой индекс создать и почему именно такой порядок полей?
```
| name | sex | year |
```
```sql
SELECT name
WHERE sex = 'F' AND year = 2000
```

## Решение
```sql
```
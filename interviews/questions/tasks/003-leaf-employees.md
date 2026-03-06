---
type: task
companies:
  - Wildberries
topic: PostgreSQL
subtopic: JOIN
title: Найти листовых сотрудников в дереве иерархии
---

## Условие


Дана таблица сотрудников компании `employee`:
- `id int` — идентификатор сотрудника
- `parent_id int` — идентификатор его руководителя
- `name string` — имя сотрудника

**Задача:** вывести имена всех линейных сотрудников, которые не являются руководителями (все листья в дереве иерархии).

## Решение

**С JOIN:**
```sql
SELECT e.name
FROM employee e
LEFT JOIN employee sub ON e.id = sub.parent_id
WHERE sub.id IS NULL;
```

**С подзапросом:**
```sql
SELECT name 
FROM employee
WHERE id NOT IN (SELECT parent_id FROM employee WHERE parent_id IS NOT NULL);
```

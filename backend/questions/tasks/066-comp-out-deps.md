---
type: task
companies:
  - MTS
topic: PostgreSQL
subtopic:
  - JOIN
  - UNION
  - GROUP BY
  - HAVING
title: Запросы по сотрудникам и департаментам — JOIN, UNION, фильтрация
---

## Условие

Даны три таблицы:

- `comp_users` (id int, depID int, name varchar, welcomDate int, active bool)
- `out_users` (id int, depID int, name varchar, active bool)
- `deps` (id int, name varchar)

Задания:
1. Написать запрос, который покажет имя сотрудника и идентификатор департамента
2. Получить всех сотрудников компании, включая внешних
3. Получить не идентификатор департамента, а его название
4. Получить только действующих сотрудников (active = true)
5. Вывести имена сотрудников и названия департаментов только тех департаментов, у которых сотрудников больше 20

## Пример
```
### in
comp_users, out_users, deps — таблицы со структурой выше

### out
-- запросы по каждому заданию
```

## Решение
```sql
```
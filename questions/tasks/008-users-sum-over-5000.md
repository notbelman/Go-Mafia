---
type: task
companies:
  - OZON
topic: PostgreSQL
subtopic:
  - GROUP BY
  - HAVING
  -  Агрегация
title: Пользователи с суммой покупок больше 5000
---

## Условие
Найти пользователей, которые совершили покупок на сумму больше 5000р.

Вывести их имена в формате: `id пользователя | имя | фамилия | сумма покупок`

## Решение

```sql
SELECT u.id, u.firstname, u.lastname, SUM(p.price)
FROM purchase p
JOIN user u ON p.user_id = u.id
GROUP BY u.id, u.firstname, u.lastname
HAVING SUM(p.price) > 5000;
```

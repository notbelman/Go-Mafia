---
type: task
companies:
  - OZON
topic: PostgreSQL
subtopic:
  - JOIN
  - DISTINCT
title: Покупки пользователей до бана
---

## Условие
Таблицы:

```sql
-- user
id | firstname | lastname | birth
1  | Ivan      | Petrov   | 1996-05-01
2  | Anna      | Petrova  | 1999-06-01
3  | Anna      | Petrova  | 1990-10-02

-- purchase
sku  | price | user_id | date
5500 | 1     | 2021-02-15
5700 | 1     | 2021-01-15
4000 | 1     | 2022-02-14
8000 | 2     | 2021-03-01
400  | 2     | 2021-03-02

-- ban_list
user_id | date_from
1       | 2021-03-08
```

Вывести уникальные комбинации пользователя и id товара для всех покупок, совершённых пользователями до того, как их забанили.

Отсортировать сначала по имени пользователя, потом по SKU.

## Решение

```sql
SELECT DISTINCT u.*, p.sku 
FROM purchase p
JOIN user u ON p.user_id = u.id
LEFT JOIN ban_list b ON p.user_id = b.user_id
WHERE b.user_id IS NULL OR p.date < b.date_from
ORDER BY u.firstname, p.sku;
```

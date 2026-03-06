---
type: task
companies:
  - Uzum
topic: PostgreSQL
subtopic:
  - GROUP BY
  - HAVING
  - Indexes
  - EXPLAIN
title: Магазины с количеством заказов за июнь больше 100
---

## Условие

Дана таблица заказов. Получите все магазины с количеством заказов за июнь больше 100. Оптимизируйте запрос.
```sql
CREATE TABLE orders (
    id         integer primary key not null,
    sum        integer             not null,
    user_id    integer             not null,
    shop_id    integer             not null,
    created_at timestamptz         not null,
    updated_at timestamptz         not null
);
```

## Решение
```sql
```
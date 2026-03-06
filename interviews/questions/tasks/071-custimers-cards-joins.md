---
type: task
companies:
  - LAMODA
topic: PostgreSQL
subtopic:
  - JOIN
  - LEFT JOIN
  - GROUP BY
  - HAVING
  - ORDER BY
  - NULL
title: Запросы по покупателям и корзине — JOIN, топ-10, фильтрация
---

## Условие

Даны две таблицы:
```sql
CREATE TABLE customer (
    id      INTEGER PRIMARY KEY,
    email   VARCHAR(100) NOT NULL,
    country CHAR(2)      NOT NULL
);

CREATE TABLE cart_item (
    id          INTEGER PRIMARY KEY,
    customer_id INTEGER     NOT NULL,
    title       VARCHAR(20) NOT NULL,
    amount      INTEGER     NOT NULL,
    price       INTEGER     NOT NULL
);
```

Данные:
- customer: (1, c1@example.com, ru), (2, c2@example.com, ru), (3, c3@example.com, ru), (4, c4@example.com, by)
- cart_item: (1, 1, пиво, 6, 100), (2, 1, вода, 2, 50), (3, 2, печенье, 1, 75), (4, 2, сок, 2, 60)

Задания:
1. Вывести построчно всех кастомеров (id, email) и все элементы корзины (title, amount)
2. Вывести топ-10 клиентов (id, email) по общей стоимости товаров в корзине
3. Покупатели из России (id, email, сумма корзины), которые положили товаров не менее чем на 1000 рублей

## Решение
```sql
```
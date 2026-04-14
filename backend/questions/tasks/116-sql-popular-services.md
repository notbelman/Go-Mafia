---
type: task
companies:
  - Napoleon IT
topic: SQL
subtopic:
  - JOIN
  - GROUP BY
  - Schema Design
title: Структура таблиц и запрос популярных услуг
---

## Условие

1. Составить структуру таблиц без типов: заказы, пользователи, услуги.
2. Количество самых популярных заказанных услуг и название где цена выше 3000 за последние 30 дней.

## Решение

### Вариант 1: одна услуга на заказ (service_id прямо в orders)

```sql
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE services (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL,
    price NUMERIC NOT NULL
);

CREATE TABLE orders (
    id         SERIAL PRIMARY KEY,
    user_id    INT NOT NULL REFERENCES users(id),
    service_id INT NOT NULL REFERENCES services(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

#### Запрос: топ-10 популярных услуг дороже 3000 за 30 дней

```sql
SELECT
    s.name,
    s.price,
    COUNT(o.id) AS order_count
FROM orders o
JOIN services s ON s.id = o.service_id
WHERE
    o.created_at >= now() - INTERVAL '30 days'
    AND s.price > 3000
GROUP BY s.id
ORDER BY order_count DESC
LIMIT 10;
```

### Вариант 2: несколько услуг в одном заказе (many-to-many)

Доп. вопрос на собесе: "одна услуга в одном заказе — нужно по-другому". Добавляем промежуточную таблицу `order_services`:

```sql
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE services (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL,
    price NUMERIC NOT NULL
);

CREATE TABLE orders (
    id         SERIAL PRIMARY KEY,
    user_id    INT NOT NULL REFERENCES users(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE order_services (
    order_id   INT NOT NULL REFERENCES orders(id),
    service_id INT NOT NULL REFERENCES services(id),
    PRIMARY KEY (order_id, service_id)
);
```

#### Запрос: топ-10 популярных услуг дороже 3000 за 30 дней

```sql
SELECT
    s.name,
    s.price,
    COUNT(os.order_id) AS order_count
FROM order_services os
JOIN services s ON s.id = os.service_id
JOIN orders o ON o.id = os.order_id
WHERE
    o.created_at >= now() - INTERVAL '30 days'
    AND s.price > 3000
GROUP BY s.id
ORDER BY order_count DESC
LIMIT 10;
```

### Доп. вопрос: почему `s.id` не нужен в SELECT?

`s.id` — суррогатный ключ, в результате бесполезен. `GROUP BY s.id` нужен чтобы PostgreSQL знал как группировать, но выводить его не обязательно — `SELECT` и `GROUP BY` не обязаны совпадать. Группируем по `s.id`, показываем только `s.name`, `s.price`, `COUNT(...)`.

### Заметки

- `GROUP BY s.id` — достаточно, потому что `s.id` — primary key, `s.name` и `s.price` функционально зависят от него. PostgreSQL это понимает.
- Индексы для варианта 2: `CREATE INDEX idx_orders_created_at ON orders(created_at)` и `CREATE INDEX idx_order_services_service_id ON order_services(service_id)`.

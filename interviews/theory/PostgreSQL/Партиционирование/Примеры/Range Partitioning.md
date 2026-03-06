```sql
-- создание партицированной таблицы
CREATE TABLE orders (
    id SERIAL,
    created_at DATE NOT NULL,
    amount NUMERIC
) PARTITION BY RANGE (created_at);

-- создание партиций
CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

-- default для строк вне диапазонов
CREATE TABLE orders_default PARTITION OF orders DEFAULT;

-- индекс на родителе - создастся на всех партициях
CREATE INDEX idx_orders_created_at ON orders (created_at);

-- вставка - автоматически роутится в orders_2024
INSERT INTO orders (created_at, amount) VALUES ('2024-06-15', 100);

-- запрос - pruning оставит только orders_2024
SELECT * FROM orders WHERE created_at = '2024-06-15';

-- удаление старых данных - мгновенно, без bloat
DROP TABLE orders_2023;
-- или отсоединить для архива
ALTER TABLE orders DETACH PARTITION orders_2023;
```
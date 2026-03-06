```sql
-- создание партицированной таблицы
CREATE TABLE orders (
    id SERIAL,
    user_id INT NOT NULL,
    amount NUMERIC
) PARTITION BY HASH (user_id);

-- создание партиций (modulus = количество партиций)
CREATE TABLE orders_p0 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE orders_p1 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE orders_p2 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE orders_p3 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);

-- индекс
CREATE INDEX idx_orders_user_id ON orders (user_id);

-- вставка - hash(123) % 4 определяет партицию
INSERT INTO orders (user_id, amount) VALUES (123, 100);

-- pruning работает ТОЛЬКО с точным равенством
SELECT * FROM orders WHERE user_id = 123;  -- одна партиция

-- pruning НЕ работает с диапазонами
SELECT * FROM orders WHERE user_id > 100;  -- full scan всех партиций

-- изменить modulus нельзя - только пересоздать все партиции
```
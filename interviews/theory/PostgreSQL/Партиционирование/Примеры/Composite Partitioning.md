```sql
-- создание партицированной таблицы (первый уровень - Range по дате)
CREATE TABLE orders (
    id SERIAL,
    user_id INT NOT NULL,
    created_at DATE NOT NULL,
    amount NUMERIC
) PARTITION BY RANGE (created_at);

-- партиция 2024, сама разбита на субпартиции (Hash по user_id)
CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01')
    PARTITION BY HASH (user_id);

-- субпартиции для 2024
CREATE TABLE orders_2024_p0 PARTITION OF orders_2024
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE orders_2024_p1 PARTITION OF orders_2024
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE orders_2024_p2 PARTITION OF orders_2024
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE orders_2024_p3 PARTITION OF orders_2024
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);

-- партиция 2025 с такой же структурой
CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01')
    PARTITION BY HASH (user_id);

CREATE TABLE orders_2025_p0 PARTITION OF orders_2025
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
-- ... p1, p2, p3

-- pruning на обоих уровнях
SELECT * FROM orders 
WHERE created_at = '2024-06-15' AND user_id = 123;
-- 1) Range pruning: только orders_2024
-- 2) Hash pruning: только orders_2024_pN

-- pruning частичный - только первый уровень
SELECT * FROM orders WHERE created_at = '2024-06-15';
-- Range pruning работает, но сканирует все 4 субпартиции

-- retention - дропаем весь год со всеми субпартициями
DROP TABLE orders_2024;
```
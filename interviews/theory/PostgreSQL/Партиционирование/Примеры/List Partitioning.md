```sql
-- создание партицированной таблицы
CREATE TABLE orders (
    id SERIAL,
    region TEXT NOT NULL,
    amount NUMERIC
) PARTITION BY LIST (region);

-- создание партиций по регионам
CREATE TABLE orders_eu PARTITION OF orders
    FOR VALUES IN ('DE', 'FR', 'IT', 'ES');

CREATE TABLE orders_us PARTITION OF orders
    FOR VALUES IN ('US', 'CA');

CREATE TABLE orders_asia PARTITION OF orders
    FOR VALUES IN ('JP', 'CN', 'KR');

-- default для неизвестных регионов
CREATE TABLE orders_other PARTITION OF orders DEFAULT;

-- индекс
CREATE INDEX idx_orders_region ON orders (region);

-- вставка - роутится в orders_eu
INSERT INTO orders (region, amount) VALUES ('DE', 100);

-- запрос - pruning оставит только orders_eu
SELECT * FROM orders WHERE region = 'DE';

-- pruning работает и с IN
SELECT * FROM orders WHERE region IN ('DE', 'FR');

-- добавление нового региона = новая партиция (DDL)
CREATE TABLE orders_latam PARTITION OF orders
    FOR VALUES IN ('BR', 'MX', 'AR');
```
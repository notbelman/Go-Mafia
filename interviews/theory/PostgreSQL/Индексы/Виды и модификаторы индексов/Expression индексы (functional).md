**Что это:** Индекс на результат выражения или функции, а не на колонку напрямую.

```sql
CREATE INDEX idx_email_lower ON users(LOWER(email));
```

**Зачем:** Обычный индекс не работает, если в WHERE колонка обёрнута в функцию.

**Правило:** Выражение в индексе должно точно совпадать с выражением в запросе.

**Когда использовать:**

- Case-insensitive поиск (`LOWER`, `UPPER`)
- Поиск по части даты (`DATE()`, `EXTRACT()`)
- Вычисляемые поля (`(data->>'field')::int`)
- Поиск по части строки (`SUBSTRING`)

**Ограничения:**

- Только IMMUTABLE функции (без `NOW()`, `random()`)
- Индекс больше обычного (хранит вычисленные значения)

---

**Примеры с кодом:**

**Проблема без expression индекса:**

```sql
CREATE INDEX idx_email ON users(email);

SELECT * FROM users WHERE LOWER(email) = 'test@mail.ru';
-- Seq Scan ❌ - индекс не используется
```

**Решение:**

```sql
CREATE INDEX idx_email_lower ON users(LOWER(email));

SELECT * FROM users WHERE LOWER(email) = 'test@mail.ru';
-- Index Scan ✅
```

**Поиск по дате:**

```sql
-- Найти все заказы за конкретный день
CREATE INDEX idx_order_date ON orders(DATE(created_at));

SELECT * FROM orders WHERE DATE(created_at) = '2024-01-15';
-- Index Scan ✅
```

**JSONB поля:**

```sql
CREATE INDEX idx_user_age ON profiles(((data->>'age')::int));

SELECT * FROM profiles WHERE (data->>'age')::int > 18;
-- Index Scan ✅
```

**Выражение должно совпадать:**

```sql
CREATE INDEX idx_lower ON users(LOWER(email));

WHERE LOWER(email) = 'x'    -- ✅ использует
WHERE UPPER(email) = 'X'    -- ❌ не использует
WHERE email = 'x'           -- ❌ не использует
```
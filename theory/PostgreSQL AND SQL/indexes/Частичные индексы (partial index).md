- **Частичный индекс** — индекс только на подмножество строк через `WHERE`. Индексируем 1% вместо 100% → экономия места в десятки раз ^pi-def
- Идеален для: булевых флагов с перекосом (99% true / 1% false), статусов (только pending), soft delete (`WHERE deleted_at IS NULL`) ^pi-when
- WHERE в запросе **должен включать** условие индекса, иначе не используется ^pi-rule

---

**Что это:** Индекс только на подмножество строк таблицы, определённое через `WHERE`.

```sql
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;
```

**Зачем:**

- Экономия места (индексируем 1% вместо 100%)
- Ускорение запросов по редким значениям
- Обход низкой селективности булевых полей

**Когда использовать:**

- Булевы флаги с перекосом (99% true, 1% false)
- Статусы (интересны только pending, не completed)
- Soft delete (`WHERE deleted_at IS NULL`)
- Свежие данные (`WHERE created_at > '2024-01-01'`)

**Правила:**

- WHERE в запросе должен включать условие индекса
- Условие должно быть immutable (без `NOW()`)

---

**Примеры с кодом:**

**Экономия места:**

```sql
-- Обычный: все 10M строк → ~3GB
CREATE INDEX idx_all ON users(email);

-- Частичный: только 100K активных → ~30MB
CREATE INDEX idx_active ON users(email) WHERE is_active = true;
```

**Булевы флаги:**

```sql
-- 99% активны, 1% забанены - индексируем редкий случай
CREATE INDEX idx_banned ON users(id) WHERE is_banned = true;
```

**Статусы:**

```sql
CREATE INDEX idx_pending ON orders(created_at) 
WHERE status IN ('pending', 'processing');
```

**Soft delete:**

```sql
CREATE INDEX idx_active_docs ON documents(updated_at) 
WHERE deleted_at IS NULL;
```

**Использование индекса:**

```sql
CREATE INDEX idx_active ON users(email) WHERE is_active = true;

-- ✅ Использует
SELECT * FROM users WHERE email = 'x@y.com' AND is_active = true;

-- ❌ Не использует (нет условия)
SELECT * FROM users WHERE email = 'x@y.com';

-- ❌ Не использует (другое значение)
SELECT * FROM users WHERE email = 'x@y.com' AND is_active = false;
```


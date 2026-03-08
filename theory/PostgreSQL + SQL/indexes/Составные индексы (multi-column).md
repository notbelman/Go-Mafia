- **Составной индекс** — индекс на несколько колонок. Порядок **критически важен**: работает только с первой колонки (**правило левого префикса**) ^ci-def
- Порядок: **equality первым**, **range последним**. `WHERE status = X AND created_at > Y` → индекс `(status, created_at)`, не наоборот ^ci-order
- Составной vs несколько одиночных: часто `WHERE A AND B` → составной. Часто `WHERE A` или `WHERE B` по отдельности → два одиночных ^ci-vs-single

---

**Что это:** Индекс на несколько колонок, можно использовать фильтрацию. Порядок колонок критически важен.

```sql
CREATE INDEX idx_name ON users(last_name, first_name, age);
```

**Правило левого префикса:** Индекс работает только с первой колонки. Пропустил первую - индекс не используется.

```
Индекс (A, B, C):
WHERE A           ✅
WHERE A AND B     ✅
WHERE A AND B AND C  ✅
WHERE C AND B AND A  ✅
WHERE C AND A AND B  ✅
WHERE B           ❌
WHERE B AND C     ❌
WHERE C           ❌
```

**Порядок колонок - как выбрать:**

1. Колонки с `=` - первыми
2. Колонки с range (`>`, `<`, `BETWEEN`) - последними
3. Чаще используется в запросах → ближе к началу

**Составной vs несколько одиночных:**

- Часто `WHERE A AND B` → составной
- Часто `WHERE A` или `WHERE B` по отдельности → два одиночных

---

**Примеры с кодом:**

**Правило левого префикса:**

```sql
-- Индекс: (last_name, first_name, age)

WHERE last_name = 'Иванов' AND first_name = 'Алексей' AND age = 25  -- ✅ полностью
WHERE last_name = 'Иванов' AND first_name = 'Алексей'               -- ✅ частично
WHERE last_name = 'Иванов'                                          -- ✅ частично
WHERE first_name = 'Алексей' AND age = 25                           -- ❌ пропущена первая
WHERE age = 25                                                       -- ❌ пропущена первая
```

**Порядок: equality перед range:**

```sql
-- Запрос: WHERE status = 'active' AND created_at > '2024-01-01'

CREATE INDEX idx_bad ON orders(created_at, status);   -- ❌ range первый
CREATE INDEX idx_good ON orders(status, created_at);  -- ✅ equality первый
```

**Выбор стратегии:**

```sql
-- Часто: WHERE last_name = X AND age = Y
CREATE INDEX idx_composite ON users(last_name, age);  -- ✅ один составной

-- Часто: WHERE last_name = X или WHERE age = Y (по отдельности)
CREATE INDEX idx_last_name ON users(last_name);       -- ✅ два одиночных
CREATE INDEX idx_age ON users(age);
```

**Как хранится:**

```
Сортировка как tuple:
('Иванов', 'Алексей', 25) → TID_1
('Иванов', 'Алексей', 30) → TID_2
('Иванов', 'Борис', 25)   → TID_3
('Петров', 'Алексей', 25) → TID_4
```

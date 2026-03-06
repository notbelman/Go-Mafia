NULL = NULL даёт NULL, не TRUE. Строки с NULL не джойнятся.
## Почему

В SQL: `NULL = NULL` → `NULL` (не TRUE и не FALSE)
JOIN оставляет строку только если условие = TRUE.

## Проблема
```sql
SELECT * FROM t1 JOIN t2 ON t1.code = t2.code;
```

t1: ('A'), ('B'), (NULL)
t2: ('A'), (NULL), (NULL)

**Результат: только ('A', 'A')**

NULL из t1 не сджойнился ни с одним NULL из t2.

## Когда это больно

- JOIN по nullable FK
- JOIN по опциональным полям (email, phone)
- Миграции с грязными данными

## Решение если нужно джойнить NULL
```sql
-- Явная обработка:
ON t1.code = t2.code OR (t1.code IS NULL AND t2.code IS NULL)

-- Или через COALESCE с меткой:
ON COALESCE(t1.code, '___NULL___') = COALESCE(t2.code, '___NULL___')
```

⚠️ Обычно NULL не джойнится — это правильное поведение. Чини данные, а не запрос.
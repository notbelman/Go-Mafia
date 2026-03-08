- **LATERAL** — модификатор JOIN: подзапрос в FROM **видит колонки** левой таблицы. Без LATERAL — ошибка компиляции
- Выполняется **для каждой строки** левой таблицы отдельно (как коррелированный подзапрос, но в FROM)
- **LEFT JOIN LATERAL**: подзапрос вернул 0 строк → строка остаётся с NULL. 
- **CROSS JOIN LATERAL**: 0 строк → строка **исчезает**

---

## Синтаксис

```sql
SELECT u.name, recent.*
FROM users u
LEFT JOIN LATERAL (
    SELECT * FROM orders o
    WHERE o.user_id = u.id    -- u виден благодаря LATERAL!
    ORDER BY o.created_at DESC
    LIMIT 3
) recent ON true;
```

Без LATERAL:
```sql
-- ❌ ОШИБКА: subquery cannot reference u
LEFT JOIN (SELECT * FROM orders WHERE user_id = u.id) ...
```

## Зачем

**TOP-N per group** — главный use case:
```sql
-- 3 последних заказа КАЖДОГО юзера
SELECT u.id, o.*
FROM users u
LEFT JOIN LATERAL (
    SELECT * FROM orders
    WHERE user_id = u.id
    ORDER BY created_at DESC
    LIMIT 3
) o ON true;
```

**Table-функция с параметрами:**
```sql
SELECT u.id, t.*
FROM users u
CROSS JOIN LATERAL unnest(u.tags) AS t(tag);
```

**Развернуть JSON:**
```sql
SELECT e.id, j.*
FROM events e
CROSS JOIN LATERAL jsonb_each(e.data) AS j(key, value);
```

## LEFT LATERAL vs CROSS LATERAL

```
LEFT JOIN LATERAL:  подзапрос вернул 0 строк → строка с NULL (сохраняется)
CROSS JOIN LATERAL: подзапрос вернул 0 строк → строка исчезает
```

Юзер без заказов:
- `LEFT JOIN LATERAL` → `(user_id=5, NULL, NULL, NULL)` — юзер в результате
- `CROSS JOIN LATERAL` → юзера нет в результате

## ON true

`LEFT JOIN LATERAL (...) sub ON true` — фильтрация уже внутри подзапроса (WHERE), ON true = "всегда соединять". Стандартный паттерн.

## Связь
- [[Типы JOIN]] — LATERAL работает с любым типом: LEFT, CROSS, INNER
- [[Производительность JOIN]] — LATERAL = Nested Loop (подзапрос на каждую строку)

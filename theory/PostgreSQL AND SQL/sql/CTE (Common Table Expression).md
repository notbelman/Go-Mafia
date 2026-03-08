- **CTE** (Common Table Expression) — именованный подзапрос через `WITH`. Улучшает читаемость, позволяет переиспользовать результат в запросе
- PG 12+: CTE **инлайнится** по умолчанию (оптимизатор проталкивает условия внутрь). `MATERIALIZED` — принудительная материализация. PG < 12: **всегда** материализуется (барьер для оптимизатора)
- **Рекурсивный CTE** (`WITH RECURSIVE`) — для иерархий (дерево категорий, оргструктура, графы). Base case + UNION ALL + рекурсивный шаг
- CTE **не замена** временной таблицы: CTE живёт **один запрос**, temp table — всю сессию

---

## Базовый CTE

```sql
WITH active_users AS (
  SELECT id, name, email
  FROM users
  WHERE status = 'active'
)
SELECT au.name, COUNT(o.id) AS order_count
FROM active_users au
JOIN orders o ON o.user_id = au.id
GROUP BY au.name;
```

Тот же результат что подзапрос, но **читаемее** при сложных запросах.

## Материализация (PG 12+)

```sql
-- PG 12+: инлайнится (оптимизатор проталкивает WHERE внутрь)
WITH cte AS (
  SELECT * FROM huge_table
)
SELECT * FROM cte WHERE id = 42;
-- → оптимизатор превратит в: SELECT * FROM huge_table WHERE id = 42

-- Принудительная материализация
WITH cte AS MATERIALIZED (
  SELECT * FROM huge_table
)
SELECT * FROM cte WHERE id = 42;
-- → сначала SELECT * FROM huge_table в temp, потом фильтрация (плохо!)

-- Принудительный инлайн
WITH cte AS NOT MATERIALIZED (
  SELECT * FROM huge_table
)
SELECT * FROM cte WHERE id = 42;
```

**PG < 12**: CTE **всегда** материализуется. `WITH` = барьер для оптимизатора → может быть **медленнее** подзапроса.

### Когда MATERIALIZED полезен

```sql
-- CTE используется ДВАЖДЫ → без материализации выполнится дважды
WITH stats AS MATERIALIZED (
  SELECT department, AVG(salary) AS avg_sal
  FROM employees
  GROUP BY department
)
SELECT * FROM stats WHERE avg_sal > 100000
UNION ALL
SELECT * FROM stats WHERE avg_sal < 50000;
```

## Рекурсивный CTE

```sql
-- Дерево категорий: найти все подкатегории
WITH RECURSIVE tree AS (
  -- Base case: корневая категория
  SELECT id, name, parent_id, 1 AS depth
  FROM categories
  WHERE id = 1
  
  UNION ALL
  
  -- Рекурсивный шаг: дети текущего уровня
  SELECT c.id, c.name, c.parent_id, t.depth + 1
  FROM categories c
  JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree;
```

```
Итерация 1: id=1 (корень)
Итерация 2: id=2, id=3 (дети корня)
Итерация 3: id=4, id=5 (дети id=2 и id=3)
...пока UNION ALL не вернёт 0 строк → стоп
```

### Защита от бесконечной рекурсии

```sql
WITH RECURSIVE tree AS (
  SELECT id, parent_id, 1 AS depth FROM categories WHERE id = 1
  UNION ALL
  SELECT c.id, c.parent_id, t.depth + 1
  FROM categories c
  JOIN tree t ON c.parent_id = t.id
  WHERE t.depth < 10  -- ограничение глубины
)
SELECT * FROM tree;
```

### Оргструктура: все подчинённые

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, name, manager_id FROM employees WHERE id = 1  -- CEO
  UNION ALL
  SELECT e.id, e.name, e.manager_id
  FROM employees e
  JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

### Графы: кратчайший путь (BFS)

```sql
WITH RECURSIVE path AS (
  SELECT id, ARRAY[id] AS visited, 0 AS depth
  FROM nodes WHERE id = 'A'
  
  UNION ALL
  
  SELECT e.to_node, p.visited || e.to_node, p.depth + 1
  FROM edges e
  JOIN path p ON e.from_node = p.id
  WHERE NOT (e.to_node = ANY(p.visited))  -- защита от циклов
    AND p.depth < 20
)
SELECT * FROM path WHERE id = 'Z' ORDER BY depth LIMIT 1;
```

## CTE vs подзапрос vs temp table

| | CTE | Подзапрос | Temp table |
|:--|:--|:--|:--|
| Живёт | один запрос | один запрос | вся сессия |
| Инлайнится (PG 12+) | да | да | нет (физическая таблица) |
| Рекурсия | да | нет | нет |
| Индексы | нет | нет | можно создать |
| Читаемость | лучше | хуже при вложенности | отдельно |

## CTE для DML (writeable CTE)

```sql
-- Удалить и вернуть удалённые строки
WITH deleted AS (
  DELETE FROM expired_sessions
  WHERE expires_at < NOW()
  RETURNING *
)
INSERT INTO session_archive SELECT * FROM deleted;
```

## Связь
- [[Вспомогательные ноды EXPLAIN]] — CTE Scan в плане запроса
- [[Производительность JOIN]] — CTE как барьер оптимизатора (PG < 12)
- [[Оконные функции]] — CTE + оконные функции = мощная комбинация

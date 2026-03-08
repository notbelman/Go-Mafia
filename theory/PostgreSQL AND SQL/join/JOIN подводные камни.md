- **ON vs WHERE в LEFT JOIN**: фильтр правой таблицы — в **ON** (до добавления сирот). В WHERE — **превращает LEFT в INNER** (отрезает NULL)
- **NULL = NULL → NULL**, не TRUE. Строки с NULL **не джойнятся**. Это правильное поведение — чини данные, не запрос
- **Диаграммы Венна врут**: JOIN ≠ пересечение множеств. JOIN **умножает** совпавшие строки (2×2=4)
- `USING (id)` тоже самое, что и `t1.id = t2.td`
-  **NATURAL JOIN** — автоматический USING по всем одноимённым колонкам. Добавил колонку → сломал запрос. **Не использовать в проде.**

---

## ON vs WHERE в LEFT JOIN

ON применяется **ДО** добавления сирот. WHERE — **ПОСЛЕ** (отрезает NULL).

```sql
-- ON: строки без пары ОСТАЮТСЯ (NULL справа)
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'paid';
-- юзеры без paid-заказов → строка с NULL

-- WHERE: строки без пары ИСЧЕЗАЮТ (NULL != 'paid')
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'paid';
-- юзеры без paid-заказов → ИСЧЕЗЛИ. LEFT стал INNER.
```

**Правило**: фильтры правой таблицы → ON. Фильтры левой → WHERE.

## NULL в условиях JOIN

```sql
SELECT * FROM t1 JOIN t2 ON t1.code = t2.code;

-- t1: ('A'), ('B'), (NULL)
-- t2: ('A'), (NULL), (NULL)
-- Результат: только ('A', 'A'). NULL не сджойнились.
```

`NULL = NULL` → NULL (не TRUE) → строка не проходит ON.

Если **реально нужно** джойнить NULL (редко):
```sql
ON t1.code = t2.code OR (t1.code IS NULL AND t2.code IS NULL)
-- или
ON COALESCE(t1.code, '___NULL___') = COALESCE(t2.code, '___NULL___')
```

## Почему диаграммы Венна врут

```
t1: {1, 1, 3}
t2: {1, 1, 2}

Ожидание по кругам: 2 строки (пересечение)
Реальность INNER JOIN: 4 строки (2×2)
```

- Таблица ≠ множество (дубликаты!)
- JOIN ≠ INTERSECT (это разные операции)
- **JOIN = CROSS JOIN + фильтр = умножение, не пересечение**

## Условие ON — не только id = id

```sql
-- Пересечение диапазонов
JOIN cities_ip_ranges c ON c.ip_range && s.ip

-- BETWEEN
JOIN prices ON date BETWEEN start_date AND end_date

-- Всегда TRUE = CROSS JOIN
JOIN t2 ON true
```

99% разработчиков думают что ON только для `id = id`.

## USING и NATURAL

```sql
-- Эквивалентны:
SELECT * FROM t1 JOIN t2 ON t1.id = t2.id;
SELECT * FROM t1 JOIN t2 USING (id);
```

USING убирает дубликат колонки из `SELECT *`.

**NATURAL JOIN** — автоматический USING по всем одноимённым колонкам. Добавил колонку → сломал запрос. **Не использовать в проде.**

## Связь
- [[Типы JOIN]] — как работают INNER, LEFT, RIGHT, FULL, CROSS
- [[LATERAL JOIN]] — ON true как стандартный паттерн
- [[Производительность JOIN]] — ON vs WHERE влияет на план

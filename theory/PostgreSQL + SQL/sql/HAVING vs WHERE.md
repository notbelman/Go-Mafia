- **WHERE** — фильтрует **строки до** GROUP BY. **HAVING** — фильтрует **группы после** GROUP BY. Разные этапы выполнения
- В HAVING можно использовать **агрегатные функции** (COUNT, SUM, AVG). В WHERE — **нельзя**
- Порядок: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`
- Правило: если условие **не содержит агрегат** — пиши в WHERE (эффективнее, меньше строк для группировки)

---

## Порядок выполнения SQL

```
FROM        → выбрать таблицы
WHERE       → отфильтровать СТРОКИ              ← до группировки
GROUP BY    → сгруппировать
HAVING      → отфильтровать ГРУППЫ              ← после группировки
SELECT      → вычислить выражения, оконные функции
ORDER BY    → отсортировать
LIMIT       → обрезать
```

## Пример

```sql
-- Найти отделы с более чем 5 сотрудниками старше 30
SELECT department, COUNT(*) AS cnt
FROM employees
WHERE age > 30           -- фильтр СТРОК (до GROUP BY)
GROUP BY department
HAVING COUNT(*) > 5;     -- фильтр ГРУПП (после GROUP BY)
```

**WHERE** `age > 30` отсеял строки → GROUP BY сгруппировал оставшиеся → **HAVING** `COUNT(*) > 5` отсеял мелкие группы.

## Типичная ошибка

```sql
-- ❌ ОШИБКА: aggregate functions are not allowed in WHERE
SELECT department, COUNT(*)
FROM employees
WHERE COUNT(*) > 5
GROUP BY department;

-- ✅ ПРАВИЛЬНО: агрегат в HAVING
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

## Когда что использовать

| Условие | Куда | Почему |
|:--|:--|:--|
| `age > 30` | WHERE | не агрегат → фильтруй **рано**, меньше строк для GROUP BY |
| `COUNT(*) > 5` | HAVING | агрегат → только после GROUP BY |
| `status = 'active'` | WHERE | не агрегат |
| `SUM(amount) > 1000` | HAVING | агрегат |
| `department = 'IT'` | WHERE | **не** HAVING (хотя сработает, но неэффективно) |

**Антипаттерн**: писать неагрегатное условие в HAVING — PG сначала сгруппирует **все** строки, потом отфильтрует группы. В WHERE отфильтровал бы строки **до** группировки → меньше работы.

## HAVING без GROUP BY

```sql
-- Есть ли хоть один сотрудник старше 60?
SELECT COUNT(*) FROM employees HAVING COUNT(*) > 0 AND MAX(age) > 60;
-- вся таблица = одна группа
```

Редкий случай, но валидный SQL.

## Связь
- [[GROUP BY и индексы]] — GROUP BY + индекс = GroupAggregate без сортировки
- [[Оконные функции]] — оконные функции после HAVING, до ORDER BY
- [[EXPLAIN основы]] — Filter vs HAVING в плане запроса

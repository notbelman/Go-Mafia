- **INNER** = CROSS JOIN + фильтр. **LEFT** = INNER + строки без пары (NULL). **RIGHT** = зеркало LEFT. **FULL** = LEFT + RIGHT. **CROSS** = декартово произведение
- JOIN при дубликатах **умножает**: 2 строки слева × 2 справа = **4 строки**. Это НЕ пересечение множеств
- LEFT JOIN может вернуть **больше строк** чем в левой таблице (из-за дубликатов справа)
- RIGHT JOIN **не нужен** — всегда можно переписать как LEFT, поменяв таблицы. FULL JOIN в MySQL **нет** — эмуляция через UNION

---

## Общий пример

```
t1: (1), (1), (3)
t2: (1), (1), (4), (5)
```

## INNER JOIN

```sql
SELECT * FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

| t1.id | t2.id |
|---|---|
| 1 | 1 |
| 1 | 1 |
| 1 | 1 |
| 1 | 1 |

**4 строки, не 2.** Каждая 1 слева × каждая 1 справа = 2×2 = 4.

Эквивалентно:
```sql
SELECT * FROM t1 CROSS JOIN t2 WHERE t1.id = t2.id;
```

## LEFT JOIN

```sql
SELECT * FROM t1 LEFT JOIN t2 ON t1.id = t2.id;
```

| t1.id | t2.id |
| ----- | ----- |
| 1     | 1     |
| 1     | 1     |
| 1     | 1     |
| 1     | 1     |
| 3     | NULL  |

INNER + строка 3 без пары → NULL справа. **5 строк из 3 в левой таблице.**

## RIGHT JOIN

```sql
SELECT * FROM t1 RIGHT JOIN t2 ON t1.id = t2.id;
```

| t1.id | t2.id |
|---|---|
| 1 | 1 |
| 1 | 1 |
| 1 | 1 |
| 1 | 1 |
| NULL | 4 |
| NULL | 5 |

Зеркало LEFT. Всегда можно переписать как LEFT, поменяв таблицы.

## FULL OUTER JOIN

```sql

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

| t1.id | t2.id |
| ----- | ----- |
| 1     | 1     |
| 1     | 1     |
| 1     | 1     |
| 1     | 1     |
| 3     | NULL  |
| NULL  | 4     |
| NULL  | 5     |

LEFT + RIGHT вместе. В MySQL нет — эмулировать через `UNION ALL`.

## CROSS JOIN

```sql
SELECT * FROM t1 CROSS JOIN t2;
-- или: SELECT * FROM t1, t2;
```

Все комбинации: 3 × 4 = 12 строк. **Фундамент всех JOIN'ов.**

## Сводка

```
CROSS JOIN           → все × все
INNER JOIN           → CROSS + фильтр ON
LEFT [OUTER] JOIN    → INNER + сироты слева (NULL справа)
RIGHT [OUTER] JOIN   → INNER + сироты справа (NULL слева)
FULL [OUTER] JOIN    → INNER + сироты с обеих сторон
```

## Связь
- [[JOIN подводные камни]] — ON vs WHERE, NULL, диаграммы Венна
- [[LATERAL JOIN]] — подзапрос видит колонки левой таблицы
- [[Производительность JOIN]] — Nested Loop, Hash, Merge, anti/semi-join

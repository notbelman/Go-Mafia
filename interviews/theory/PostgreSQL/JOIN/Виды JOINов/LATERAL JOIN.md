Модификатор JOIN — подзапрос в FROM видит колонки левой таблицы.

Работает с любым типом: `CROSS JOIN LATERAL`, `LEFT JOIN LATERAL`, `INNER JOIN LATERAL`

## Зачем

- TOP-N per group (LIMIT на каждую строку)
- Вызов table-функции с параметрами из строки
- Развернуть JSON/массив с контекстом строки

## Синтаксис
```sql
SELECT *
FROM t1
LEFT JOIN LATERAL (
    SELECT ... FROM t2 WHERE t2.x = t1.y  -- t1 виден!
) sub ON true;
```

Без LATERAL подзапрос не видит t1 — ошибка.
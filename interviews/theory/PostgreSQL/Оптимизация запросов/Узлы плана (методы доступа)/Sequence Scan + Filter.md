Seq Scan читает ВСЕ строки, Filter (WHERE) отбрасывает ненужные после чтения.
```
Seq Scan on t1  (actual rows=10)
  Filter: (col = 'x')
  Rows Removed by Filter: 99990  ← прочитали 100к, оставили 10
```

## Rows Removed by Filter

- Мало → норм
- Много → нужен индекс, читаем лишнее

Решение: `CREATE INDEX ON t1(col);`
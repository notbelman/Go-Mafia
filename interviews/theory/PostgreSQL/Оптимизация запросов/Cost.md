cost — условные единицы, не секунды
```
cost = (pages_read × page_cost) + (rows × cpu_tuple_cost)
```

- `seq_page_cost = 1.0` — последовательное чтение страницы
- `random_page_cost = 4.0` — случайное чтение (индекс)
- `cpu_tuple_cost = 0.01` — обработка строки

Где page_cost:
- `seq_page_cost = 1.0` — если Seq Scan
- `random_page_cost = 4.0` — если Index Scan (прыгает по диску)

Поэтому Index Scan на много строк может быть дороже Seq Scan — случайные чтения ×4.
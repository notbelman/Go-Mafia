Поиск через индекс → достаём строки из heap. Каждая строчка читается рандомно
```
Index Scan using users_pkey on users  (cost=0.29..8.30 rows=1)
  Index Cond: (id = 42)
```

## Когда выбирается

- Мало строк (<5-10% таблицы)
- Есть подходящий индекс

## Стоимость
```
cost = (index_pages × random_page_cost) + 
       (rows × cpu_index_tuple_cost) +
       (rows × random_page_cost) +      ← поход в heap за каждой строкой
       (rows × cpu_tuple_cost)
```

## Почему может быть дороже Seq Scan

Каждая строка = random I/O в heap. 30% таблицы через индекс = много random чтений vs один проход Seq Scan.

⚠️ На SSD можно снизить `random_page_cost` (дефолт 4.0, для SSD ~1.1)
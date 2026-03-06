Кэширование результата узла, чтобы не вычислять повторно. Появляется внутри Nested Loop.

## Когда появляется

- Внутренний узел выполняется много раз (loops > 1)
- Результат небольшой и влезает в work_mem

## В EXPLAIN
```
Nested Loop
  ->  Seq Scan on orders (actual rows=1000 loops=1)
  ->  Materialize (actual rows=50 loops=1000)  ← кэш перечитан 1000 раз (чем больше, тем лучше)
        ->  Seq Scan on users (actual rows=50 loops=1)  ← выполнился 1 раз
              Filter: (age > 30)
```
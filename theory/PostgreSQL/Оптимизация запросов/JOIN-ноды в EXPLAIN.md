- **Nested Loop** — для каждой строки внешней ищем во внутренней. O(N × log M) с индексом, **O(N × M) без** = катастрофа. Единственный вариант для `<`, `>`, `BETWEEN`
- **Hash Join** — строим хэш-таблицу из меньшей (Build), пробегаем по большей (Probe). **O(N + M)**. Только `=`. `Batches > 1` = не влезло в work_mem → диск
- **Merge Join** — обе стороны отсортированы, идём параллельно. **O(N + M)** если есть индекс, O(N log N + M log M) без. Только `=`
- **Anti Join** = `NOT EXISTS` (нет ни одного совпадения). **Semi Join** = `EXISTS` (есть хотя бы одно). Модификаторы к любой стратегии
- **Materialize** — кэширует результат внутреннего узла чтобы не вычислять повторно в Nested Loop

---

## Nested Loop Join

```
Nested Loop  (cost=0.29..850.00)
  ->  Seq Scan on users u              ← внешняя: 10 строк
        Filter: (status = 'vip')
  ->  Index Scan on orders o           ← внутренняя: выполнится 10 раз
        Index Cond: (user_id = u.id)
        (actual rows=5 loops=10)       ← 5 строк × 10 раз = 50 мс
```

| Ситуация | Сложность |
|:--|:--|
| С индексом на внутренней | O(N × log M) |
| Unique index (1 строка) | O(N) |
| **Без индекса** | **O(N × M)** — катастрофа |

**Единственный вариант** для неравенств (`<`, `>`, `BETWEEN`). Hash и Merge не умеют.

**Хорош с LIMIT** — может остановиться рано (не нужно читать всё).

## Hash Join

```
Hash Join  (cost=30.00..150.00 rows=1000)
  Hash Cond: (o.user_id = u.id)
  ->  Seq Scan on orders o         ← Probe: пробегаем по большей
  ->  Hash                         ← Build: хэш-таблица из меньшей
        Batches: 1                 ← всё в памяти ✅
        Memory Usage: 512kB
        ->  Seq Scan on users u
```

**Batches > 1** = хэш-таблица не влезла в work_mem → часть на диске → **медленно**:
```
->  Hash (rows=10000)
      Batches: 8    ← часть на диске
      Memory Usage: 4096kB
```
Решение: `SET work_mem = '256MB';`

Только `=`. Не отдаёт результат до конца build-фазы (плохо с LIMIT).

## Merge Join

```
Merge Join  (cost=200.00..350.00)
  Merge Cond: (u.id = o.user_id)
  ->  Index Scan using users_pkey on users     ← уже отсортирован ✅
  ->  Index Scan using idx_orders on orders    ← уже отсортирован ✅
```

Идеально когда обе стороны уже отсортированы (по индексу). Без индексов:
```
Merge Join
  ->  Sort                      ← дорого
        ->  Seq Scan on users
  ->  Sort                      ← ещё дороже
        ->  Seq Scan on orders
```

Только `=`. Выбирается когда хэш-таблица не влезет в memory (большие таблицы).

## Anti Join / Semi Join

Модификаторы к **любой** стратегии (Nested Loop / Hash / Merge):

```
Hash Anti Join       ← NOT EXISTS: нет ни одного совпадения
Nested Loop Semi Join ← EXISTS: есть хотя бы одно совпадение
```

```sql
-- Anti Join
SELECT * FROM t1 WHERE NOT EXISTS (SELECT 1 FROM t2 WHERE t2.id = t1.id);

-- Semi Join
SELECT * FROM t1 WHERE EXISTS (SELECT 1 FROM t2 WHERE t2.id = t1.id);
```

Semi Join останавливается на **первом** найденном совпадении — эффективнее обычного JOIN.

## Materialize

Кэширует результат узла чтобы не вычислять повторно в Nested Loop:

```
Nested Loop
  ->  Seq Scan on orders (actual rows=1000 loops=1)
  ->  Materialize (actual rows=50 loops=1000)  ← кэш перечитан 1000 раз
        ->  Seq Scan on users (actual rows=50 loops=1)  ← выполнился 1 раз
              Filter: (age > 30)
```

Появляется когда внутренний узел маленький и выполняется много раз.

## Сводка

| | Nested Loop | Hash Join | Merge Join |
|:--|:--|:--|:--|
| Условие | любое (=, <, >) | только = | только = |
| Сложность | O(N × log M) | O(N + M) | O(N + M) |
| Нужен индекс | на внутренней (желательно) | нет | нет (но сортировка) |
| Нужна память | нет | work_mem (хэш) | work_mem (сортировка) |
| LIMIT | хорош | плох | плох |
| Большие таблицы | плох без индекса | хорош | хорош |

## Связь
- [[Производительность JOIN]] — когда какой, оптимизация, anti/semi в SQL
- [[EXPLAIN основы]] — loops × actual time
- [[EXPLAIN красные флаги]] — Batches > 1, Nested Loop без индекса

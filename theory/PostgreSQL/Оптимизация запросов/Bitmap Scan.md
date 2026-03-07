- **Bitmap Scan** — двухфазная операция для средней селективности (**5-20%**). Фаза 1: Bitmap Index Scan строит битовую карту страниц. Фаза 2: Bitmap Heap Scan читает страницы **последовательно**
- Превращает **random I/O в sequential** — Index Scan прыгает хаотично, Bitmap читает страницы по порядку номеров
- **BitmapAnd** / **BitmapOr** — комбинирование нескольких индексов: `WHERE a=1 AND b=2` → BitmapAnd, `WHERE a=1 OR b=2` → BitmapOr
- **Exact bitmap** (влез в work_mem) — знает конкретные строки. **Lossy** (не влез) — знает только страницы → Recheck проверяет **все** строки на странице
- `Heap Blocks: lossy=1200` → bitmap не влез → увеличь `work_mem`

---

## Как работает

```
Bitmap Heap Scan on orders              ← фаза 2: читаем страницы по порядку
  Recheck Cond: (amount > 1000)         ← перепроверяем строки
  ->  Bitmap Index Scan on idx_amount   ← фаза 1: строим битовую карту
        Index Cond: (amount > 1000)
```

**Фаза 1 (Bitmap Index Scan):** проходим по индексу, отмечаем страницы и строки в битовой карте.

**Фаза 2 (Bitmap Heap Scan):** читаем отмеченные страницы **последовательно** (по номерам). Recheck проверяет какие строки реально подходят.

## Exact vs Lossy

**Exact** (bitmap влез в work_mem) — битовый массив по строкам:
```
страница 3: [1,0,0,1,0,0,0,0...]  ← точно: строки 1, 4
страница 5: [0,0,1,0,0,0,1,0...]  ← точно: строки 3, 7
```
Recheck проверяет только помеченные строки.

**Lossy** (не влез) — схлопывается до страниц:
```
[0,0,1,0,1,0,0...]  ← страницы 3, 5 помечены, какие строки — неизвестно
```
Recheck проверяет **все** строки на странице — медленнее.

**Диагностика:**
```
Heap Blocks: exact=500 lossy=0     ← ОК
Heap Blocks: exact=0 lossy=1200    ← bitmap не влез → SET work_mem = '256MB'
```

## BitmapAnd / BitmapOr

Комбинирование нескольких индексов через битовые операции:

```sql
WHERE status = 'active' AND amount > 1000
```

```
Bitmap Heap Scan on orders
  ->  BitmapAnd
        ->  Bitmap Index Scan on idx_status    ← bitmap 1
        ->  Bitmap Index Scan on idx_amount    ← bitmap 2
                                               ← AND → пересечение
```

```sql
WHERE status = 'active' OR amount > 1000
```

```
Bitmap Heap Scan on orders
  ->  BitmapOr
        ->  Bitmap Index Scan on idx_status
        ->  Bitmap Index Scan on idx_amount
```

**BitmapAnd vs составной индекс**: составной обычно быстрее (один проход). Но BitmapOr — часто **единственный** способ для OR с разными колонками.

## Когда Bitmap Scan

```
             Seq Scan         Bitmap Scan        Index Scan
Строк        >20%             5-20%              <5%
I/O          sequential       sequential (!)     random
Порядок      физический       по номерам стр.    хаотичный
```

Bitmap — **золотая середина**: слишком много строк для Index Scan (random дорого), слишком мало для Seq Scan.

## Связь
- [[Scan-ноды в EXPLAIN]] — Seq Scan и Index Scan для сравнения
- [[EXPLAIN основы]] — как читать Heap Blocks в BUFFERS
- [[BRIN индекс]] — BRIN тоже использует Bitmap Heap Scan
- [[EXPLAIN красные флаги]] — lossy bitmap = красный флаг

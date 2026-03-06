Комбинирование нескольких индексов через битовые операции.
```
Bitmap Heap Scan on orders
  ->  BitmapAnd
        ->  Bitmap Index Scan on idx_status
        ->  Bitmap Index Scan on idx_amount
```

## Когда

- `WHERE a = 1 AND b = 2` → BitmapAnd
- `WHERE a = 1 OR b = 2` → BitmapOr

## vs составной индекс

Составной индекс обычно быстрее BitmapAnd — один проход вместо двух.

BitmapOr — часто единственный способ для OR.
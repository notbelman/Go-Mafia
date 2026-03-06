---
type: task
companies:
  - GroupIB
topic: PostgreSQL
subtopic:
  - Composite Index
  - Index Selection
title: Выбор одного индекса для нескольких запросов
---

## Условие

Есть таблица T со столбцами a, b, c. Нужно создать один индекс, который будет эффективно применяться в каждом из запросов.

## Пример
```sql
SELECT * FROM T WHERE a = A AND b = B AND c = C;
SELECT * FROM T WHERE b = B;
SELECT * FROM T WHERE a > A AND b = B AND c = C;
SELECT * FROM T WHERE c = C AND a = A AND b = B;
```

## Решение
```sql
```
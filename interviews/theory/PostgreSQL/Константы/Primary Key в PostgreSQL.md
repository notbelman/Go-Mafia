PK = NOT NULL + UNIQUE. Одна таблица — один PK.

## Что делает под капотом

- Автоматически создаёт **unique B-tree индекс** (руками создавать не нужно, дубликат — вредно)
- Гарантирует уникальность + запрет NULL
- Используется для **logical replication** (PostgreSQL идентифицирует строки по PK)
- Оптимизатор использует PK для выбора плана выполнения
- Некоторые инструменты (pg_repack, DMS, DataGrip) требуют PK для работы

## Составной PK
```sql
CREATE TABLE order_items (
    order_id INT,
    item_no INT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, item_no)
);
```

- Уникальность по **комбинации**: `(1, 1)` и `(1, 2)` — ок, два `(1, 1)` — нет
- ОБЕ колонки NOT NULL (даже в составном PK нельзя NULL ни в одной)
- Индекс составной — работает для запросов по `order_id` или `(order_id, item_no)`, но НЕ по `item_no` отдельно (leftmost prefix правило)


| | PRIMARY KEY | UNIQUE |
|:--|:-----------|:-------|
| NULL | Запрещён | Допускает несколько NULL |
| Сколько на таблицу | Один | Сколько угодно |
| Индекс | Unique B-tree | Unique B-tree |
| Logical replication | Да, по умолчанию | Нужно указать вручную |
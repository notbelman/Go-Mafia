- **B+Tree**: запись = random I/O (перезапись страницы). Чтение = **1 seek**. Используют PG, MySQL, Oracle
- **LSM-Tree**: запись = sequential I/O (MemTable → SSTable → compaction). Чтение = **проверка нескольких SSTable**. Используют RocksDB, Cassandra
- Правило: **много чтений** → B+Tree, **много записей** → LSM-Tree

---

## Как работают

**B+Tree** — сбалансированное дерево, данные в leaf-страницах, обновления in-place.

**LSM-Tree** — пишет в память (MemTable), при заполнении flush на диск как SSTable. Периодически compaction.
```
B+Tree:  запись → найти страницу → перезаписать на диске (random I/O)
LSM:    запись → MemTable (RAM) → flush → SSTable (sequential I/O) → compaction
```

## Сравнение

|                  | B+Tree                          | LSM-Tree                          |
| :--------------- | :------------------------------ | :-------------------------------- |
| Запись           | Медленнее (random I/O)          | Быстрее (sequential I/O)         |
| Чтение           | Быстрее (1 seek по дереву)      | Медленнее (несколько SSTable) |
| Обновление       | In-place                        | Append-only |
| Write amplification | Высокий | Ниже |
| Read amplification  | Низкий | Выше (bloom-фильтры помогают) |
| Compaction        | Не нужен | Обязателен, жрёт CPU и I/O |

## Где используются

| B+Tree | LSM-Tree |
|:--|:--|
| PostgreSQL, MySQL (InnoDB), Oracle | RocksDB, LevelDB, Cassandra, HBase |
| Read-heavy, OLTP, транзакции | Write-heavy, логи, time-series |

## Связь
- [[theory/PostgreSQL AND SQL/indexes/B-tree индекс]] — внутреннее устройство B-tree в PG
- [[BRIN индекс]] — альтернатива B-tree для append-only данных

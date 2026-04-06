## Быстрая навигация

- [[Индексы]] — все типы, оптимизация, мониторинг (24 файла)
- [[Транзакции]] — ACID, MVCC, изоляция, блокировки (11 файлов)
- [[Оптимизация]] — EXPLAIN, scan/join ноды, work_mem (7 файлов)
- [[SQL]] — CTE, оконные функции, триггеры, миграции (7 файлов)
- [[Партиционирование]] — типы, pruning, pg_partman (6 файлов)
- [[JOIN]] — типы, LATERAL, подводные камни, производительность (4 файла)
- [[Ограничения]] — PK, FK, UNIQUE, CHECK, модификаторы (5 файлов)
- [[Репликация]] — топологии, WAL, sync/async, CDC (2 файла)
- [[Шардирование]] — стратегии, consistent hashing, vs репликация (3 файла)
- [[Vacuum]] — VACUUM vs VACUUM FULL (2 файла)

---

## [[Индексы]]

### Типы индексов
- [[Какие бывают индексы в PostgreSQL - сводка]]
- [[B-tree индекс]]
- [[Hash индекс]]
- [[GIN индекс]]
- [[GiST индекс]]
- [[BRIN индекс]]
- [[B-Tree vs LSM-Tree]]

### Оптимизация
- [[Составные индексы (multi-column)]]
- [[Частичные индексы (partial index)]]
- [[Покрывающие индексы (covering index Index-Only Scan)]]
- [[Expression индексы (functional)]]
- [[Уникальные индексы и constraints]]
- [[Селективность и издержки индексов]]
- [[Сложность операций индексов]]
- [[Group by и индексы]]

### Внутреннее устройство
- [[Heap страницы и TID]]
- [[HOT updates]]
- [[Dead tuples и bloat]]
- [[pg_trgm и поиск подстроки]]

### Мониторинг
- [[Мониторинг индексов]]

---

## [[Транзакции]]

### Основы
- [[Транзакции и ACID]]
- [[MVCC в PostgreSQL]]
- [[Уровни изоляции PostgreSQL]]

### Аномалии
- [[Dirty Read (грязное чтение)]]
- [[Non-Repeatable Read (неповторяющееся чтение)]]
- [[Phantom Read (фантомное чтение)]]
- [[Lost Update (потерянное обновление)]]

### Блокировки
- [[Блокировки PostgreSQL]]
- [[Advisory locks]]
- [[SSI (Serializable Snapshot Isolation)]]

### Идемпотентность
- [[Exactly-once — доставка vs обработка]]

---

## [[Оптимизация]]

- [[EXPLAIN основы]]
- [[EXPLAIN красные флаги]]
- [[Scan-ноды в EXPLAIN]]
- [[Bitmap Scan]]
- [[JOIN-ноды в EXPLAIN]]
- [[Вспомогательные ноды EXPLAIN]]
- [[work_mem]]

---

## [[SQL]]

- [[CTE (Common Table Expression)]]
- [[Оконные функции]]
- [[HAVING vs WHERE]]
- [[Триггеры PostgreSQL]]
- [[Миграции без даунтайма]]
- [[Write amplification]]
- [[SELECT all антипаттерн]]

---

## [[Партиционирование]]

- [[Партиционирование]] — обзор, pruning, выбор ключа
- [[Range Partitioning]]
- [[List Partitioning]]
- [[Hash Partitioning]]
- [[Composite Partitioning]]
- [[pg_partman]]

---

## [[JOIN]]

- [[Типы JOIN]]
- [[JOIN подводные камни]]
- [[LATERAL JOIN]]
- [[Производительность JOIN]]

---

## [[Ограничения]]

- [[Общая табличка]]
- [[Primary Key в PostgreSQL]]
- [[Уникальные индексы и constraints]]
- [[FOREIGN KEY действия]]
- [[Модификаторы]]

---

## [[Репликация]]

- [[Репликация]] — топологии, sync/async, failover, WAL
- [[Репликация — детали]] — lag, аномалии, CDC, libslave

---

## [[Шардирование]]

- [[Шардирование]] — стратегии, shard key, cross-shard, rebalancing
- [[Consistent Hashing и Virtual Buckets]]
- [[Partitioning vs Sharding vs Replication]]

---

## [[Vacuum]]

- [[VACUUM]]
- [[VACUUM FULL]]

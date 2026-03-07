- **MVCC не всегда достаточно**: DDL (ALTER TABLE), явная координация, предотвращение lost update без retry → нужны **явные блокировки**
- **8 уровней** table-level locks: от ACCESS SHARE (SELECT) до ACCESS EXCLUSIVE (DROP, TRUNCATE, VACUUM FULL). Совместимость — матрица конфликтов
- **SELECT FOR UPDATE** — блокирует **конкретные строки** (row-level lock). `FOR NO KEY UPDATE`, `FOR SHARE`, `FOR KEY SHARE` — более гранулярные варианты
- **Deadlock**: PG обнаруживает через wait-for graph после `deadlock_timeout` (**1s дефолт**). Убивает одну транзакцию: `ERROR: deadlock detected`
- Диагностика: `pg_locks` + `pg_stat_activity` + `pg_blocking_pids()`

---

## Зачем явные блокировки

MVCC не всегда достаточно:
- **DDL операции** (ALTER TABLE) — нужен ACCESS EXCLUSIVE
- **Явная координация** — "хочу изменить эту строку, остальные ждите"
- **Предотвращение lost update** без retry — SELECT FOR UPDATE вместо REPEATABLE READ + retry

## Table-level locks (8 уровней)

| Режим | Когда | Блокирует |
|:--|:--|:--|
| ACCESS SHARE | SELECT | только ACCESS EXCLUSIVE |
| ROW SHARE | SELECT FOR UPDATE/SHARE | EXCLUSIVE, ACCESS EXCLUSIVE |
| ROW EXCLUSIVE | INSERT, UPDATE, DELETE | SHARE и выше |
| SHARE UPDATE EXCLUSIVE | VACUUM, ANALYZE, CREATE INDEX CONCURRENTLY | сам себя и выше |
| SHARE | CREATE INDEX | ROW EXCLUSIVE и выше |
| SHARE ROW EXCLUSIVE | CREATE TRIGGER | ROW EXCLUSIVE и выше |
| EXCLUSIVE | REFRESH MAT VIEW CONCURRENTLY | ROW SHARE и выше |
| **ACCESS EXCLUSIVE** | **DROP, TRUNCATE, ALTER, VACUUM FULL** | **всё** |

**Ключевое**: SELECT берёт ACCESS SHARE → не конфликтует с INSERT/UPDATE/DELETE (ROW EXCLUSIVE). Читатели не мешают писателям — это MVCC.

## Row-level locks (SELECT FOR ...)

```sql
-- Блокировать строки для обновления
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- другие транзакции ЖДУТ на этой строке

-- Варианты:
FOR UPDATE              -- самый жёсткий: блокирует для UPDATE/DELETE
FOR NO KEY UPDATE       -- как FOR UPDATE, но не блокирует FK-проверки
FOR SHARE               -- разделяемая: несколько FOR SHARE совместимы
FOR KEY SHARE           -- самый лёгкий: только FK-проверки
```

**SKIP LOCKED** — пропустить заблокированные строки (очередь задач):
```sql
SELECT * FROM tasks WHERE status = 'pending'
ORDER BY created_at
LIMIT 1
FOR UPDATE SKIP LOCKED;
-- если строка заблокирована другой транзакцией → берём следующую
```

**NOWAIT** — не ждать, сразу ошибка:
```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- ERROR: could not obtain lock on row (если занята)
```

## Deadlock

```
T1: SELECT * FROM accounts WHERE id=1 FOR UPDATE;  ← получил лок на id=1
T2: SELECT * FROM accounts WHERE id=2 FOR UPDATE;  ← получил лок на id=2
T1: SELECT * FROM accounts WHERE id=2 FOR UPDATE;  ← ЖДЁТ T2
T2: SELECT * FROM accounts WHERE id=1 FOR UPDATE;  ← ЖДЁТ T1 → DEADLOCK
```

PG обнаруживает через **wait-for graph** после `deadlock_timeout` (дефолт **1 секунда**). Убивает одну транзакцию:
```
ERROR: deadlock detected
DETAIL: Process 1234 waits for ShareLock on transaction 5678;
        blocked by process 5678.
        Process 5678 waits for ShareLock on transaction 1234;
        blocked by process 1234.
```

**Решение**: всегда блокировать строки в **одном порядке** (ORDER BY id).

```sql
-- ХОРОШО: обе транзакции блокируют id=1 первым
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
```

## Диагностика блокировок

```sql
-- Кто кого блокирует
SELECT
  blocked.pid AS blocked_pid,
  blocked.query AS blocked_query,
  blocking.pid AS blocking_pid,
  blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
  ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE blocked.pid != blocked.pid;

-- Все текущие блокировки
SELECT locktype, relation::regclass, mode, granted, pid
FROM pg_locks
WHERE NOT granted;  -- ожидающие блокировки
```

## Связь
- [[MVCC в PostgreSQL]] — MVCC = основа, явные блокировки = дополнение
- [[Lost Update (потерянное обновление)]] — SELECT FOR UPDATE предотвращает
- [[Deadlock]] — deadlock в Go (аналогия: порядок захвата мьютексов)
- [[Advisory locks]] — пользовательские блокировки
- [[2PL]] — 2PL = абстрактный алгоритм, PG locks = реализация

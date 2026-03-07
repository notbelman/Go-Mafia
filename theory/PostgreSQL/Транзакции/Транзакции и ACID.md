- **Транзакция** — группа операций: либо **все** успешно, либо **ни одна**. BEGIN → операции → COMMIT (фиксация) или ROLLBACK (откат)
- **ACID**: **A**tomicity (всё или ничего), **C**onsistency (инварианты не нарушены), **I**solation (параллельные транзакции не видят промежуточных состояний), **D**urability (после COMMIT данные не потеряются)
- После ошибки внутри транзакции PG переходит в состояние **"aborted"** — любые команды кроме ROLLBACK **игнорируются**
- **SAVEPOINT** — точка сохранения: `ROLLBACK TO savepoint` откатывает часть транзакции, не всю
- **Главный компромисс**: полная изоляция дорогая → уровни изоляции жертвуют частью ради производительности

---

## ACID

| Буква | Свойство | Суть | Механизм в PG |
|:--|:--|:--|:--|
| **A** | Atomicity | Всё или ничего. Один запрос упал → вся транзакция откатывается | WAL (Write-Ahead Log) |
| **C** | Consistency | БД переходит из одного корректного состояния в другое | Constraints (UNIQUE, FK, CHECK, NOT NULL) |
| **I** | Isolation | Параллельные транзакции не влияют друг на друга | MVCC + уровни изоляции |
| **D** | Durability | После COMMIT данные сохранены навсегда, даже если сервер упадёт | WAL, fsync, реплики |

## Синтаксис

```sql
BEGIN;
  UPDATE account SET balance = balance - 100 WHERE id = 1;
  UPDATE account SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

```sql
-- При ошибке:
BEGIN;
  UPDATE account SET balance = balance - 100 WHERE id = 1;
  INSERT INTO transfers(from_id, to_id) VALUES (1, 999);  -- FK violation!
  -- транзакция в состоянии "aborted"
  -- ЛЮБЫЕ команды кроме ROLLBACK → ERROR: current transaction is aborted
ROLLBACK;  -- обязательно!
```

## SAVEPOINT

```sql
BEGIN;
  UPDATE account SET balance = 500 WHERE id = 1;
  
  SAVEPOINT my_point;
  UPDATE account SET balance = -100 WHERE id = 2;  -- ошибка (CHECK constraint)
  ROLLBACK TO my_point;  -- откат до savepoint, НЕ всей транзакции
  
  UPDATE account SET balance = 200 WHERE id = 2;  -- ОК
COMMIT;  -- оба UPDATE зафиксированы
```

Вложенные SAVEPOINT'ы работают как стек: можно делать SAVEPOINT внутри SAVEPOINT.

## Autocommit

В PG каждый отдельный запрос без BEGIN — **неявная транзакция** (autocommit). `UPDATE users SET ...` без BEGIN = BEGIN → UPDATE → COMMIT автоматически.

## Связь
- [[Уровни изоляции PostgreSQL]] — компромисс I из ACID
- [[MVCC в PostgreSQL]] — как PG реализует Isolation
- [[VACUUM]] — Durability через WAL, VACUUM чистит последствия MVCC

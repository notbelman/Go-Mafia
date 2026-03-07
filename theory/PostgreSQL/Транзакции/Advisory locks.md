- **Advisory locks** — пользовательские блокировки в PG. Не привязаны к таблицам/строкам — блокируешь произвольный **числовой ключ** (int64). Приложение само решает семантику
- Два типа: **session-level** (живёт до disconnect / явного unlock) и **transaction-level** (автоматически с COMMIT/ROLLBACK)
- Use cases: **distributed locking** (один инстанс обрабатывает задачу), **миграции** (один деплой мигрирует), **cron-задачи** (не запускать параллельно)
- **Не забывай разблокировать** session-level! Каждый `pg_advisory_lock` требует парного `pg_advisory_unlock`. Без пары — лок живёт до конца сессии

---

## Синтаксис

```sql
-- Session-level (живёт до unlock или disconnect)
SELECT pg_advisory_lock(12345);       -- блокирующий (ждёт)
SELECT pg_try_advisory_lock(12345);   -- неблокирующий (true/false)
SELECT pg_advisory_unlock(12345);     -- разблокировать

-- Transaction-level (автоматически с COMMIT/ROLLBACK)
SELECT pg_advisory_xact_lock(12345);
-- не нужен unlock — освободится при COMMIT/ROLLBACK
```

## Два ключа

```sql
-- Один int8 ключ
SELECT pg_advisory_lock(12345);

-- Два int4 ключа (namespace + id)
SELECT pg_advisory_lock(1, 42);  -- "тип сущности 1, id 42"
```

Два ключа удобны для **именованных ресурсов**: первый = тип (1=user, 2=order), второй = id.

## Use case: обработка задачи одним воркером

```go
func processTask(db *sql.DB, taskID int64) error {
    // Попробовать захватить лок
    var acquired bool
    err := db.QueryRow("SELECT pg_try_advisory_lock($1)", taskID).Scan(&acquired)
    if !acquired {
        return nil  // другой воркер уже обрабатывает
    }
    defer db.Exec("SELECT pg_advisory_unlock($1)", taskID)

    // Обработка задачи — гарантированно один воркер
    return doWork(taskID)
}
```

## Use case: миграции

```go
const migrationLockID = 123456789

func migrate(db *sql.DB) error {
    // Только один инстанс мигрирует
    _, err := db.Exec("SELECT pg_advisory_lock($1)", migrationLockID)
    if err != nil { return err }
    defer db.Exec("SELECT pg_advisory_unlock($1)", migrationLockID)

    return runMigrations(db)
}
```

## Use case: cron без дубликатов

```sql
-- В начале cron-задачи:
SELECT pg_try_advisory_lock(hash_of('daily_report'));
-- true → мы единственные, выполняем
-- false → другой инстанс уже работает, пропускаем
```

## Shared vs Exclusive

```sql
-- Exclusive (дефолт): только один держатель
SELECT pg_advisory_lock(123);

-- Shared: несколько читателей, эксклюзив блокирует всех
SELECT pg_advisory_lock_shared(123);
```

## Мониторинг

```sql
SELECT locktype, objid, pid, mode, granted
FROM pg_locks
WHERE locktype = 'advisory';
```

## Подводные камни

- **Session-level**: забыл `pg_advisory_unlock` → лок висит до конца сессии. С connection pool'ом (pgbouncer) сессия может жить **очень долго**
- **Connection pool**: session-level advisory locks + pgbouncer в transaction mode = **лок теряется** при возврате коннекта в пул. Используй **transaction-level** (`pg_advisory_xact_lock`) с pgbouncer
- **Не масштабируется**: advisory locks — внутри одного PG-инстанса. Для распределённого locking нужен Redis/etcd/Consul

## Связь
- [[Блокировки PostgreSQL]] — advisory locks = дополнение к table/row locks
- [[Single Flight]] — аналогичная задача: дедупликация обработки
- [[Worker Pool]] — advisory lock как альтернатива SKIP LOCKED для очереди задач

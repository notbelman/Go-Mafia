- **MVCC**: каждая модификация создаёт **новую версию** строки, не перезаписывает старую. Читатели **не блокируют** писателей, писатели **не блокируют** читателей
- Каждая версия строки хранит **xmin** (кто создал) и **xmax** (кто удалил/заменил). UPDATE = DELETE старой + INSERT новой → **дороже** чем в других СУБД
- **Snapshot** = срез БД через 3 числа: `xmin` (все до — завершены), `xmax` (все после — не начались), `xip[]` (активные между ними). Видимость: вижу то, что **закоммичено до** моего snapshot
- **XID** — 32-битный ID транзакции. Read-only получают лёгкий **vxid** (без расхода). Wraparound: 4 млрд транзакций → **VACUUM freeze** или база уходит в **read-only**
- Следствие MVCC: таблица **распухает** от старых версий → нужен VACUUM

---

## Как работает

| Операция | Что происходит в heap |
|:--|:--|
| INSERT | Новая версия с `xmin = текущий XID` |
| DELETE | Старая версия помечается `xmax = текущий XID` |
| UPDATE | DELETE старой + INSERT новой (две операции!) |

```
Heap (таблица):
┌─────┬─────┬────┬─────────┐
│xmin │xmax │ id │ balance │
├─────┼─────┼────┼─────────┤
│ 100 │ 105 │  1 │ 1000    │  ← старая версия, "удалена" T105
│ 105 │  -  │  1 │ 2000    │  ← актуальная версия
└─────┴─────┴────┴─────────┘
```

**xmin** — XID транзакции, **создавшей** эту версию.
**xmax** — XID транзакции, **удалившей/заменившей** (0 или null если актуальна).

## Snapshot и Visibility

Snapshot — срез состояния БД: какие транзакции считать завершёнными.

```
xmin = 100              -- все XID < 100 точно завершены
xmax = 110              -- все XID >= 110 ещё не начались
xip  = [102, 105, 108]  -- активные между xmin и xmax
```

**Строка видна если:**
1. `xmin` завершена и закоммичена (создатель закоммитил)
2. `xmax` либо пуста, либо не завершена, либо откачена (удалитель не закоммитил)

**Простое правило:** вижу то, что было закоммичено до моего snapshot.

### Когда создаётся snapshot

| Уровень изоляции | Когда |
|:--|:--|
| READ COMMITTED | Перед **каждым** SELECT (новый snapshot!) |
| REPEATABLE READ | На **первом запросе**, живёт до конца транзакции |
| SERIALIZABLE | На **первом запросе**, живёт до конца транзакции |

### Пример: READ COMMITTED vs REPEATABLE READ

```
READ COMMITTED:                     T2:
BEGIN;
SELECT * FROM accounts;             
-- snapshot #1: xmin=100             
                                    UPDATE accounts SET balance=2000;
                                    COMMIT;
SELECT * FROM accounts;             
-- snapshot #2: xmin=101 (НОВЫЙ!)
-- ВИДИТ изменения T2

REPEATABLE READ:                    T2:
BEGIN;
SELECT * FROM accounts;
-- snapshot зафиксирован
                                    UPDATE accounts SET balance=2000;
                                    COMMIT;
SELECT * FROM accounts;
-- ТОТ ЖЕ snapshot
-- НЕ видит изменения T2
```

## XID (Transaction ID)

32-битный идентификатор. Хранится в каждой версии строки (xmin/xmax).

| XID | Значение |
|:--|:--|
| 0 | Невалидный |
| 1 | Системная инициализация |
| 2 | **FrozenXID** — видим всем всегда |
| 3+ | Обычные транзакции |

**Virtual XID**: read-only транзакции не тратят настоящий XID — получают лёгкий vxid (только в памяти). Настоящий XID выдаётся при первом INSERT/UPDATE/DELETE.

### Wraparound

32 бита = ~4 млрд транзакций. При переполнении старые данные станут "невидимыми".

**VACUUM freeze** — заменяет старые xmin на FrozenXID (2). Строка видна **всем** навсегда.

```sql
-- Проверить возраст
SELECT datname, age(datfrozenxid) FROM pg_database;
-- age > 200M → autovacuum должен сработать
-- age > 2B   → ПАНИКА → read-only mode
```

## Мониторинг

```sql
-- Активные транзакции и их snapshot'ы
SELECT pid, xact_start, state, backend_xid, backend_xmin
FROM pg_stat_activity
WHERE state != 'idle';
-- backend_xid: XID этого backend (NULL если read-only)
-- backend_xmin: xmin snapshot'а (как далеко "держит" VACUUM)
```

## Связь
- [[MVCC]] — абстрактный алгоритм MVCC (Балун): CAS-коммит, 2PL vs MVCC
- [[VACUUM]] — чистит dead tuples, freeze XID
- [[Уровни изоляции PostgreSQL]] — когда создаётся snapshot
- [[Dead tuples и bloat]] — следствие MVCC: таблица распухает
- [[Heap страницы и TID]] — где физически хранятся версии строк

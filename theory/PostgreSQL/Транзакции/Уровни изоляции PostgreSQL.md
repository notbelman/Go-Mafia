- PG имеет **3 уровня** (READ UNCOMMITTED = READ COMMITTED). Дефолт: **READ COMMITTED**. PG **строже** стандарта SQL: фантомы закрыты уже на REPEATABLE READ
- **READ COMMITTED**: новый snapshot на **каждый SELECT**. 
  **REPEATABLE READ**: один snapshot на **всю транзакцию**. 
  **SERIALIZABLE**: snapshot + **predicate locks** (SSI)
- Выше изоляция → больше **serialization failures** → нужен **retry loop** в приложении
- **Длинные транзакции** на REPEATABLE READ/SERIALIZABLE — держат snapshot → блокируют VACUUM → таблица распухает. Правило: **не держи транзакции открытыми долго**

---

## Таблица защиты

| Уровень | Dirty Read | Lost Update | Non-Repeatable | Phantom | Write Skew |
|:--|:--|:--|:--|:--|:--|
| **READ COMMITTED** (дефолт) | ✅ | ❌ | ❌ | ❌ | ❌ |
| **REPEATABLE READ** | ✅ | ✅ | ✅ | ✅* | ❌ |
| **SERIALIZABLE** | ✅ | ✅ | ✅ | ✅ | ✅ |

*PG строже стандарта SQL — фантомы закрыты на REPEATABLE READ благодаря MVCC.

## Механизм каждого уровня

| Уровень | Snapshot | Конфликт при UPDATE | Доп. механизм |
|:--|:--|:--|:--|
| READ COMMITTED | Новый на каждый SELECT | Ждёт завершения T1, потом **перечитывает** строку | — |
| REPEATABLE READ | Один на всю транзакцию | `ERROR: could not serialize access` → **retry** | First-updater-wins |
| SERIALIZABLE | Один на всю транзакцию | `ERROR: could not serialize access` → **retry** | SSI (predicate locks) |

## READ COMMITTED (дефолт)

Каждый SELECT видит **свежий** snapshot. Между двумя SELECT'ами данные могут измениться.

**Особенность при UPDATE**: если строка изменена другой транзакцией — PG **ждёт** завершения той транзакции, потом **перечитывает** строку и проверяет WHERE заново. Поэтому Lost Update **возможен**.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- или ничего (дефолт)
```

## REPEATABLE READ

Snapshot фиксируется на **первом запросе**. Все SELECT'ы видят один и тот же срез.

При конфликте UPDATE: `ERROR: could not serialize access due to concurrent update`. Приложение должно **retry**.

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

## SERIALIZABLE

Как REPEATABLE READ + **SSI** (Serializable Snapshot Isolation). Отслеживает rw-зависимости через predicate locks. Ловит **write skew**.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

## Производительность

| Уровень | Цена |
|:--|:--|
| READ COMMITTED | Минимальная, нет serialization failures |
| REPEATABLE READ | Snapshot живёт дольше, **блокирует VACUUM**, возможны failures |
| SERIALIZABLE | Predicate locks + память, **больше retry** |

## Длинные транзакции = проблема

Snapshot на REPEATABLE READ/SERIALIZABLE **держит** `backend_xmin`. VACUUM не может удалить версии строк новее этого xmin → dead tuples копятся → **таблица распухает**.

```sql
-- Найти длинные транзакции
SELECT pid, age(clock_timestamp(), xact_start) AS duration, state, query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY xact_start;
```

**Правило**: транзакция должна жить **секунды**, не минуты. Если нужна долгая обработка — разбей на мелкие транзакции.

## Связь
- [[MVCC в PostgreSQL]] — snapshot как механизм изоляции
- [[SSI (Serializable Snapshot Isolation)]] — механизм уровня SERIALIZABLE
- [[Dirty Read (грязное чтение)]] — решается на READ COMMITTED
- [[Lost Update (потерянное обновление)]] — решается на REPEATABLE READ
- [[VACUUM]] — длинные транзакции блокируют VACUUM

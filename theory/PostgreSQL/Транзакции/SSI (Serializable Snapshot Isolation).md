- **SSI** (Serializable Snapshot Isolation) — механизм уровня SERIALIZABLE. Ловит аномалии, которые **Snapshot Isolation пропускает** (write skew)
- Отслеживает **rw-зависимости**: T1 читала данные → T2 записала в тот же диапазон → ребро T1→T2. **Цикл** (T1→T2→T1) = abort одной транзакции
- **SIREAD locks** — "мягкие" блокировки: **не ждут**, просто запоминают "кто что читал". Не путать с обычными блокировками
- Откат вместо ожидания → **лучше concurrency** чем 2PL, но бывают **false positives** (лишние откаты). Нужен **retry loop** в приложении
- **READ ONLY** транзакции не участвуют в конфликтах → дёшево на SERIALIZABLE

---

## Проблема: Write Skew

Snapshot Isolation (REPEATABLE READ) **не ловит** write skew:

```
БД: doctor_X = on_duty, doctor_Y = on_duty
Правило: минимум 1 врач на смене

T1: SELECT count(*) WHERE on_duty → 2    -- читает X и Y
T1: UPDATE SET off_duty WHERE name='Alice'

T2: SELECT count(*) WHERE on_duty → 2    -- читает X и Y (тот же snapshot!)
T2: UPDATE SET off_duty WHERE name='Bob'

T1: COMMIT ✅
T2: COMMIT ✅

Результат: 0 врачей на смене. Нарушение!
```

SI пропускает: T1 и T2 **меняют разные строки** → нет write-write конфликта. Но конфликт между **чтением** T1 и **записью** T2.

## Как SSI это ловит

SSI отслеживает **rw-зависимости** (read-write):

```
T1 читала doctor_Y → T2 записала doctor_Y  =  ребро T1 → T2
T2 читала doctor_X → T1 записала doctor_X  =  ребро T2 → T1

Граф: T1 → T2 → T1 = ЦИКЛ → abort одной транзакции
```

```
ERROR: could not serialize access due to read/write dependencies among transactions
```

## SIREAD locks

**Не настоящие блокировки** — никто не ждёт. PG просто **запоминает**: "транзакция T1 читала строки с predicate `on_duty = true`".

При записи PG проверяет: "кто-то читал этот диапазон?" → если да → добавить rw-ребро → если цикл → abort.

Три уровня гранулярности:
- **Tuple** — конкретная строка
- **Page** — страница (если много строк)
- **Relation** — вся таблица (если Seq Scan)

Чем грубее → больше false positives (лишних откатов).

## Ключевые свойства

- **Откат вместо ожидания** → лучше concurrency чем 2PL (никто не блокируется)
- **False positives** возможны — PG может откатить транзакцию которая реально не конфликтует (грубая гранулярность SIREAD)
- **READ ONLY** транзакции **не участвуют** в конфликтах → на SERIALIZABLE почти бесплатно
- Нужен **retry loop** в приложении:

```go
for {
    err := db.RunInTx(ctx, &sql.TxOptions{Isolation: sql.LevelSerializable}, func(tx *sql.Tx) error {
        // бизнес-логика
    })
    if isSerializationError(err) {
        continue  // retry
    }
    return err
}
```

## Когда использовать SERIALIZABLE

- Сложные бизнес-правила с **перекрёстными зависимостями** (финансы, бронирования)
- Проще **retry** чем городить сложные блокировки
- Когда write skew **опасен** для бизнеса

## Когда НЕ использовать

- Write-heavy нагрузка с частыми конфликтами → слишком много retry
- Простые CRUD → READ COMMITTED достаточно
- Длинные транзакции → predicate locks копятся, память растёт

## Связь
- [[Уровни изоляции PostgreSQL]] — SSI = механизм уровня SERIALIZABLE
- [[MVCC в PostgreSQL]] — SSI работает поверх snapshot'ов
- [[MVCC]] — write skew и SI vs SSI (абстрактно, Балун)
- [[Блокировки PostgreSQL]] — SIREAD != обычные блокировки

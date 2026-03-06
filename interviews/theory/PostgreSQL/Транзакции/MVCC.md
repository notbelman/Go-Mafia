- MVCC — оптимистичный concurrency control. Каждая транзакция получает снапшот (по числовому ID). Несколько версий одного ключа хранятся одновременно
- Коммит: захватить фиктивные лочки (CAS), проверить что никто не помешал (exist_between), зафиксировать. Иначе — ретрай
- Snapshot Isolation != Serializable. Write skew (пересечение чтения и записи) не решается без SSI

---

## Идея: снапшот через число

У каждой транзакции — числовой ID. Транзакция видит только данные с ID <= своего.

```
TxID=1: set(key1, "v1")
TxID=2: set(key2, "v2")
TxID=4: set(key1, "v10")     <- key1 теперь имеет ДВЕ версии
TxID=5: set(key4, "v5")

Транзакция с TxID=3: видит key1="v1", key2="v2"
                      НЕ видит key1="v10" (TxID=4 > 3) и key4 (TxID=5 > 3)
```

**Multiversion**: один ключ может храниться N раз (разные версии). Старые версии = мусор -> нужен GC (vacuum в Postgres).

## Два ID транзакции

**Read TxID** — при создании транзакции. Определяет снапшот (что видит).

**Write TxID** — при коммите, **строго после** захвата фиктивных лочек. Иначе другая транзакция может увидеть незафиксированные данные.

## Архитектура (vs 2PL)

```
2PL:  set/get -> scheduler -> блокировка -> storage (на каждую операцию)
MVCC: set -> локальный modified map (без обращения к storage!)
      get -> проверить modified -> проверить кэш -> scheduler -> storage
      commit -> scheduler -> лочки + проверка + запись в storage
      rollback -> ничего (в storage ничего не писали)
```

Транзакция копит изменения локально. При коммите — записывает всё разом.

## Коммит (оптимистичный)

```go
func (s *Scheduler) Commit(txID int, modified map[string]string) error {
    locks := collectLocks(modified)

    // 1. Захват фиктивных лочек (CAS: 0 -> txID)
    if !lockAll(locks, txID) {
        return ErrConflict  // кто-то коммитит те же ключи -> ретрай
    }
    defer unlockAll(locks)

    // 2. Взять write TxID (ПОСЛЕ лочек!)
    writeTxID := atomic.AddInt64(&s.txID, 1)

    // 3. Проверить: между read TxID и write TxID никто не менял эти ключи?
    for key := range modified {
        if s.storage.ExistBetween(key, txID, writeTxID) {
            return ErrConflict  // кто-то успел раньше -> ретрай
        }
    }

    // 4. Записать в storage
    s.storage.Set(writeTxID, modified)
    return nil
}
```

Аналогия с Git: ветка -> работаешь -> merge -> конфликт? -> resolve/retry.

## Write Skew (искажающая запись)

Snapshot Isolation **не решает** write skew:

```
БД: doctor_X = on_duty(1), doctor_Y = on_duty(1)
Правило: минимум 1 врач на дежурстве

T1: if X==1 -> set Y=0    (отпустить Y)
T2: if Y==1 -> set X=0    (отпустить X)

Результат: X=0, Y=0 -> оба врача ушли!
```

Проблема: конфликт между **чтением** одной транзакции и **записью** другой. SI проверяет только конфликты записей.

**Serializable Snapshot Isolation (SSI)** — добавляет фиктивные лочки на чтение. Значительно сложнее.

Oracle долго заявлял "serializable", хотя реализовал SI (без SSI).

## 2PL vs MVCC

| | 2PL | MVCC |
|---|---|---|
| Подход | Блокирующий (пессимистичный) | Оптимистичный |
| Пересекающиеся данные | Лучше (блокировка дешевле ретрая) | Хуже (частые ретраи) |
| Непересекающиеся данные | Хуже (лишние блокировки) | Лучше (нет блокировок) |
| Deadlock | Возможен | Невозможен (CAS + ретрай) |
| Мусор | Нет | Да (нужен vacuum/GC) |

## Связь
- [[Транзакция и ACID]] — SI решает dirty/non-repeatable/phantom read, но не write skew
- [[2PL]] — альтернативный подход
- [[CAS паттерны]] — фиктивные лочки на CAS
- [[RCU]] — похожая идея: читатели работают со снапшотом, писатель атомарно подменяет

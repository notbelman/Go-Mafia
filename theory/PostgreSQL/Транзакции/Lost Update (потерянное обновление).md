- **Lost Update** — две транзакции читают одно значение, обе меняют — одно изменение **теряется**. Классический `balance = 1000 + 10` и `balance = 1000 + 5` → итог 1005 вместо 1015
- Защита: **REPEATABLE READ+** (PG выкинет ошибку `could not serialize access`) или **SELECT FOR UPDATE** (явная блокировка строки)
- Аналог **data race с инкрементом** из Go: read-modify-write не атомарно

---

| Суть       | Две транзакции читают одно значение, обе меняют - одно изменение теряется                                  |
| ---------- | ---------------------------------------------------------------------------------------------------------- |
| Проблема   | Итоговый результат не учитывает одно из изменений                                                          |
| Защита     | REPEATABLE READ+ или SELECT FOR UPDATE                                                                     |
| PostgreSQL | На REPEATABLE READ вторая транзакция получит ошибку: `could not serialize access due to concurrent update` |

## Пример
```
balance = 1000

T1: BEGIN
T1: SELECT balance → 1000
T1: -- хочет добавить 10
                                    T2: BEGIN
                                    T2: SELECT balance → 1000
                                    T2: -- хочет добавить 5
T1: UPDATE balance = 1010
T1: COMMIT
                                    T2: UPDATE balance = 1005
                                    T2: COMMIT

Итог: balance = 1005, а не 1015! Изменения T1 потеряны.
```

## Решение через SELECT FOR UPDATE

```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;  -- блокирует строку
-- T2 ждёт здесь пока T1 не закоммитит
UPDATE accounts SET balance = balance + 10 WHERE id = 1;
COMMIT;
```

## Связь
- [[Блокировки PostgreSQL]] — SELECT FOR UPDATE как решение
- [[Уровни изоляции PostgreSQL]] — REPEATABLE READ детектит lost update
- [[Race Condition]] — аналог check-then-act в Go

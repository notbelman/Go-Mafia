
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
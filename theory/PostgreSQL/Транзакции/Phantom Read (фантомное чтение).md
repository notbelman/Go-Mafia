- **Phantom Read** — два одинаковых SELECT возвращают **разное количество строк**. Между ними другая транзакция сделала INSERT или DELETE — появился/исчез "фантом"
- По стандарту SQL защита только на **SERIALIZABLE**. Но PostgreSQL защищает уже на **REPEATABLE READ** благодаря MVCC (snapshot фиксирован)
- Отличие от Non-Repeatable Read: там **значения** существующих строк меняются (UPDATE), тут — **количество строк** (INSERT/DELETE)

---

| Суть                      | Два одинаковых SELECT возвращают разное количество строк                         |
| ------------------------- | -------------------------------------------------------------------------------- |
| Проблема                  | Появляются/исчезают "фантомные" строки во время транзакции                       |
| Отличие от Non-Repeatable | Там меняются значения существующих строк, тут - количество строк (INSERT/DELETE) |
| Защита по стандарту       | SERIALIZABLE                                                                     |
| PostgreSQL                | Защищён уже на REPEATABLE READ благодаря MVCC                                    |

## Пример
```
T1: BEGIN
T1: SELECT count(*) FROM users → 2
                                    T2: BEGIN
                                    T2: INSERT INTO users VALUES (3, 'new')
                                    T2: COMMIT
T1: SELECT count(*) FROM users → 3 (появилась "фантомная" строка!)
T1: COMMIT
```

## Связь
- [[Non-Repeatable Read (неповторяющееся чтение)]] — UPDATE vs INSERT/DELETE
- [[Уровни изоляции PostgreSQL]] — PG строже стандарта на REPEATABLE READ
- [[MVCC в PostgreSQL]] — snapshot фиксирован → фантомы не видны

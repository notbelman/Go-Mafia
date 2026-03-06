Полная перестройка, **возвращает место на диск**.

```sql
VACUUM FULL accounts;
```

||VACUUM|VACUUM FULL|
|---|---|---|
|Блокировки|нет|ACCESS EXCLUSIVE|
|Возврат места|нет|да|
|Downtime|нет|да|

Использовать редко. Альтернатива без блокировок - `pg_repack`.

**Важно:** VACUUM FULL перестраивает и heap, и индексы (с PostgreSQL 9.0+).
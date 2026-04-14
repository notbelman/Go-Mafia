---
type: task
companies:
  - Wildberries
topic: PostgreSQL
subtopic:
  - Isolation Levels
  - Transactions
  - Anomalies
  - Serializable
title: Анализ двух параллельных транзакций — аномалии на разных уровнях изоляции
---

## Условие

Есть 2 параллельных подключения к БД. На строке №2 результат селекта = 1. Какой будет результат выполнения селекта в подключении 1 на строке №6?

### Сценарий 1 (Read Committed)
```sql
Connection 1                    Connection 2
1. START TRANSACTION;
2. SELECT a FROM t WHERE i = 1;
3. ............................ START TRANSACTION;
4. ............................ UPDATE t SET a = 2 WHERE i = 1;
5. ............................ COMMIT TRANSACTION;
6. SELECT a FROM t WHERE i = 1;
7. COMMIT TRANSACTION;
```

### Сценарий 2 (Serializable)
```sql
Connection 1                    Connection 2
1. START TRANSACTION; // serializable
2. SELECT a FROM t WHERE i = 1; // A
3. ............................ START TRANSACTION;
4. ............................ UPDATE t SET a = 2 WHERE i = 1;
5. ............................ COMMIT TRANSACTION;
6. SELECT a FROM t WHERE i = 1; // B
7. COMMIT TRANSACTION;
8. SELECT a FROM t WHERE i = 1;
// А на строке №8?
```

## Решение
```
```
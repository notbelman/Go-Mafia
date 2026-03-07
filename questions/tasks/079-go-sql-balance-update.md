---
type: task
companies:
  - LAMODA
topic: Go
subtopic:
  - Transactions
  - Race Condition
  - Isolation Levels
  - SELECT FOR UPDATE
title: Анализ функции снятия денег с баланса — проблемы конкурентности
---

## Условие

Дана таблица пользователей и функция снятия денег с баланса. Разбери код, найди проблемы и предложи решения.
```sql
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY,
    balance REAL    NOT NULL
);
```
```go
func withdrawBalance(userId int32, amount float32) float32 {
    tx := BeginTransaction("ISOLATION LEVEL READ COMMITTED")

    oldBalance := tx.exec("SELECT balance FROM users WHERE user_id = $1", userId)
    if oldBalance >= amount {
        tx.exec("UPDATE users SET balance = balance - $2 WHERE user_id = $1", userId, amount)
    }
    newBalance := tx.exec("SELECT balance FROM users WHERE user_id = $1", userId)

    tx.Commit()

    return newBalance
}
```

Вопросы:
1. Что здесь происходит и есть ли проблемы?
2. Какая аномалия возможна при конкурентных вызовах?
3. Как решить — уровень изоляции, блокировки или атомарный UPDATE?

## Решение
```go
```
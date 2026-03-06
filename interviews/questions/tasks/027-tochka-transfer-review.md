---
type: task
companies:
  - Точка
topic: Go
subtopic:
  - Code Review
  - Concurrency
  - Transactions
title: Ревью кода - сервис перевода денег между счетами
---

## Условие
Провести ревью кода. Найти проблемы.

```go
package payer

type Account struct {
    Id      int
    Name    string
    Balance float32
}

// реализация скрыта, но там что-то вроде sqlx
type AccountRepository interface {
    Find(id int) (*Account, error)
    Save(acc *Account) error
}

// реализация отправки в какой-то брокер, но может и быть http post
type EventSender interface {
    Send(event interface{}) error
}

type TransferEvent struct {
    FromId int
    ToId   int
    Amount float32
}

type AccountService struct {
    repository  AccountRepository
    eventSender EventSender
}

// вызывается из rest-контроллера
func (s *AccountService) Transfer(fromId int, toId int, amount float32) error {
    fromAcc, err := s.repository.Find(fromId)
    if err != nil {
        return err
    }
    toAcc, err := s.repository.Find(toId)
    if err != nil {
        return err
    }

    toAcc.Balance += amount
    fromAcc.Balance -= amount

    err = s.repository.Save(fromAcc)
    if err != nil {
        return err
    }
    err = s.repository.Save(toAcc)
    if err != nil {
        return err
    }

    go func() {
        s.eventSender.Send(&TransferEvent{
            FromId: fromId,
            ToId:   toId,
            Amount: amount,
        })
    }()

    return nil
}
```

## Решение

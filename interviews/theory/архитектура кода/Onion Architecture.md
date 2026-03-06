## Суть

Hexagonal говорит "используй интерфейсы". Onion добавляет: внутри бизнес-логики тоже есть слои.

## Проблема

В Hexagonal всё что внутри — один ком. Domain model, бизнес-правила, оркестрация сценариев — всё вместе.

## Решение

Три слоя внутри, зависимости только внутрь:

```
Infrastructure (postgres, http, redis)
  ↓
Application Services — оркестрация: вызвать repo, отправить event
  ↓
Domain Services — логика между entities: TransferMoney(from, to)
  ↓
Domain Model — entities, value objects, правила: order.Cancel()
```

Domain Model не импортирует ничего. Ни ORM, ни фреймворки, ни application services.

## Что даёт

Самый ценный код (бизнес-правила) в самом защищённом месте — центре. Всё остальное можно менять не трогая домен.
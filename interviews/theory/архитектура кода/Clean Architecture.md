## Суть

Onion + каждый бизнес-сценарий — отдельный Use Case с явным входом и выходом.

## Проблема

В Onion Application Service — мешок методов. `OrderAppService.Create()`, `.Cancel()`, `.Refund()` — всё в одном классе, растёт бесконечно.

## Решение

Каждый сценарий — отдельная структура:

```go
type CreateOrderUseCase struct { repo OrderRepo }

func (uc *CreateOrderUseCase) Execute(in CreateOrderInput) (CreateOrderOutput, error) {
    order := domain.NewOrder(in.UserID, in.Items)
    if err := uc.repo.Save(order); err != nil { return CreateOrderOutput{}, err }
    return CreateOrderOutput{OrderID: order.ID}, nil
}
```

Input DTO → Use Case → Output DTO. Entity никогда не уходит наружу напрямую.

## Что даёт

Каждый сценарий изолирован, тестируется отдельно, не растёт в god-object. Маппинг между слоями защищает домен от протекания деталей (JSON-теги, ORM-поля).

## Цена

Много маппинга между слоями. Для CRUD на 3 эндпоинта — оверкилл.
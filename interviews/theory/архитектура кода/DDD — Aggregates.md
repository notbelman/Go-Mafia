## Суть

Группа объектов (entities + value objects) которые меняются вместе в одной транзакции. Один из них — корень, через который идёт всё общение.

## Проблема

```go
// Без агрегата — кто угодно лезет куда угодно
order.Status = "cancelled"                    // инвариант не проверен
order.Items[0].Quantity = -5                  // мусор в данных
repo.SaveOrder(order)
repo.SaveItem(order.Items[0])                 // два сохранения, между ними может быть сбой
```

## Решение

```go
// Aggregate root — единственная точка входа
type Order struct {
    id     string
    items  []OrderItem   // внутренние объекты, снаружи недоступны
    status Status
}

// Вся логика через методы root
func (o *Order) Cancel() error {
    if o.status == Shipped { return ErrAlreadyShipped }
    o.status = Cancelled
    return nil
}

func (o *Order) AddItem(product string, qty int, price Money) error {
    if o.status != Draft { return ErrOrderLocked }
    if len(o.items) >= 50 { return ErrTooManyItems }
    o.items = append(o.items, OrderItem{...})
    return nil
}
```

## Правила

- **Маленькие агрегаты** — большой агрегат = медленные транзакции, lock contention
- **1 агрегат = 1 транзакция** — не сохранять два агрегата в одной TX
- **Между агрегатами — по ID** — не `Order.Customer`, а `Order.CustomerID`
- **Repository один на агрегат** — `OrderRepository` сохраняет Order + Items целиком, отдельного `ItemRepository` нет

## Между агрегатами — domain events

Order завершён → публикуем `OrderCompleted` → Shipping-агрегат реагирует асинхронно. Не лезем в чужой агрегат напрямую.
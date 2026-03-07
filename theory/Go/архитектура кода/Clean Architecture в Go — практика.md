## Структура папок

```
internal/
  domain/          ← entities, value objects, бизнес-правила
    order.go       ← Order entity + методы (Cancel, AddItem)
    money.go       ← Money value object
    
  usecase/         ← по файлу на сценарий
    create_order.go
    cancel_order.go
    interfaces.go  ← интерфейсы которые usecase нужны
    
  adapter/
    handler/       ← HTTP/gRPC — принимает запросы
      order_handler.go
    repository/    ← postgres/redis — хранит данные
      postgres_order.go

cmd/
  server/
    main.go        ← сборка всего
```

## Domain — ни от чего не зависит

```go
// internal/domain/order.go
type Order struct {
    id     string
    items  []Item
    status Status
}

func NewOrder(userID string, items []Item) (*Order, error) {
    if len(items) == 0 { return nil, ErrEmptyOrder }
    return &Order{id: uuid.New(), items: items, status: Draft}, nil
}

func (o *Order) Cancel() error {
    if o.status == Shipped { return ErrAlreadyShipped }
    o.status = Cancelled
    return nil
}
```

## Use Cases — по одному на сценарий

```go
// internal/usecase/interfaces.go
type OrderRepo interface {
    Save(ctx context.Context, order *domain.Order) error
    FindByID(ctx context.Context, id string) (*domain.Order, error)
}

// internal/usecase/create_order.go
type CreateOrderInput struct { UserID string; Items []ItemDTO }
type CreateOrderOutput struct { OrderID string }

type CreateOrder struct { repo OrderRepo }

func (uc *CreateOrder) Execute(ctx context.Context, in CreateOrderInput) (CreateOrderOutput, error) {
    order, err := domain.NewOrder(in.UserID, toDomainItems(in.Items))
    if err != nil { return CreateOrderOutput{}, err }
    if err := uc.repo.Save(ctx, order); err != nil { return CreateOrderOutput{}, err }
    return CreateOrderOutput{OrderID: order.ID()}, nil
}

// internal/usecase/cancel_order.go
type CancelOrder struct { repo OrderRepo }

func (uc *CancelOrder) Execute(ctx context.Context, orderID string) error {
    order, err := uc.repo.FindByID(ctx, orderID)
    if err != nil { return err }
    if err := order.Cancel(); err != nil { return err }
    return uc.repo.Save(ctx, order)
}
```

## Handler — переводит HTTP → Use Case

```go
// internal/adapter/handler/order_handler.go
type OrderHandler struct {
    create *usecase.CreateOrder
    cancel *usecase.CancelOrder
}

func (h *OrderHandler) HandleCreate(w http.ResponseWriter, r *http.Request) {
    var req CreateOrderRequest
    json.NewDecoder(r.Body).Decode(&req)
    out, err := h.create.Execute(r.Context(), toInput(req))
    // ...
}
```

Handler знает про HTTP и JSON. Use Case — не знает.

## Сборка в main.go

```go
repo := repository.NewPostgresOrderRepo(db)
createUC := &usecase.CreateOrder{repo: repo}
cancelUC := &usecase.CancelOrder{repo: repo}
handler := handler.OrderHandler{create: createUC, cancel: cancelUC}
```

DI руками, без фреймворков.

## Кто что импортирует

```
handler → usecase → domain
repository → domain
handler ✗→ repository
domain ✗→ ничего
```
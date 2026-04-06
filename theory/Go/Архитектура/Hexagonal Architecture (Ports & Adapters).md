## Суть

Бизнес-логика не знает с какой БД и через какой транспорт она работает.

## Проблема

```go
// Layered: сервис напрямую зависит от postgres
type OrderService struct { db *pgxpool.Pool }
func (s *OrderService) Create(order Order) error {
    s.db.Exec("INSERT INTO orders ...") // привязан к postgres
}
// Хочешь тест — поднимай postgres. Хочешь mongo — переписывай сервис.
```

## Решение

```go
// Сервис знает только интерфейс
type OrderService struct { repo OrderRepo }

// Интерфейс — порт
type OrderRepo interface { Save(ctx context.Context, o Order) error }

// Postgres — адаптер (один из многих)
type PostgresOrderRepo struct { db *pgxpool.Pool }

// В проде: OrderService{repo: PostgresOrderRepo{db}}
// В тесте: OrderService{repo: MockOrderRepo{}}
```

## Что даёт

Меняешь postgres на mongo — меняешь один адаптер. Сервис не трогаешь. Тесты без БД.

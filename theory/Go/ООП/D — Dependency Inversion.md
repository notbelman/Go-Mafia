Зависим от интерфейса (абстракции), не от конкретного типа. Зависимости передаём снаружи через конструктор.

**Общее правило:** всё что делает I/O → за интерфейс. Чистая логика → конкретные типы.

```go
// ❌ Плохо: сервис создаёт зависимость сам, привязан к PostgreSQL
type OrderService struct {
    db *pgx.Pool
}
func NewOrderService() *OrderService {
    pool, _ := pgx.Connect(...)  // жёсткая зависимость
    return &OrderService{db: pool}
}

// ✅ Хорошо: сервис принимает интерфейс
type OrderRepo interface {
    Save(order Order) error
    GetByID(id string) (*Order, error)
}

type OrderService struct {
    repo OrderRepo  // зависит от интерфейса
}

func NewOrderService(repo OrderRepo) *OrderService {
    return &OrderService{repo: repo}
}
```

**main собирает зависимости:**
```go
func main() {
    db := connectPostgres()
    repo := postgres.NewOrderRepo(db)    // конкретная реализация
    service := NewOrderService(repo)      // передаём через конструктор
    handler := NewHandler(service)
    http.ListenAndServe(":8080", handler)
}
```

**Что закрывать за интерфейсом:**
- БД (repo) — основной кандидат
- Внешние API (биржи, платёжки, SMS)
- Кэш (Redis)
- Очереди (Kafka)
- Файловое хранилище (S3)

**В тестах:**
```go
type MockRepo struct{}
func (m *MockRepo) Save(order Order) error { return nil }
func (m *MockRepo) GetByID(id string) (*Order, error) {
    return &Order{ID: id, Status: "active"}, nil
}

func TestOrderService(t *testing.T) {
    service := NewOrderService(&MockRepo{}) // подменили БД на мок
    // тестируем бизнес-логику без реальной БД
}
```
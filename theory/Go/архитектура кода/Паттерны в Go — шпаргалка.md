## Factory — `NewXxx()`

```go
func NewOrder(userID string, items []Item) (*Order, error) {
    if len(items) == 0 { return nil, ErrEmptyOrder }
    return &Order{id: uuid.New(), items: items, status: Draft}, nil
}
```

Приватные поля + валидация в конструкторе. Не паттерн, а Go-идиома.

## Functional Options — конфигурация без боли

```go
type Option func(*Server)
func WithPort(p int) Option { return func(s *Server) { s.port = p } }
func WithTimeout(t time.Duration) Option { return func(s *Server) { s.timeout = t } }

server := NewServer(WithPort(8080), WithTimeout(5*time.Second))
```

Когда у объекта много опциональных параметров.

## Middleware/Decorator — обёртка поведения

```go
func Logging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Println(r.Method, r.URL.Path)
        next.ServeHTTP(w, r)  // вызываем следующий
    })
}

// Цепочка: Logging → Auth → Handler
router.Handle("/api", Logging(Auth(myHandler)))
```

Каждый middleware делает одну вещь и передаёт дальше.

## Strategy — интерфейс + реализации

```go
type Notifier interface { Send(msg string) error }
type EmailNotifier struct{...}
type SlackNotifier struct{...}
// Подставляем нужную реализацию без изменения вызывающего кода
```

По сути это и есть порты/адаптеры из Hexagonal.

## Observer — pub/sub через каналы

```go
events := make(chan OrderEvent, 100)
// publisher: events <- OrderCompleted{ID: "123"}
// subscriber: event := <-events
```

Domain events между агрегатами — тот же Observer.

## Table-driven tests

```go
tests := []struct {
    name  string
    input int
    want  int
}{
    {"positive", 5, 25},
    {"zero", 0, 0},
    {"negative", -3, 9},
}
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got := Square(tt.input)
        if got != tt.want { t.Errorf("got %d, want %d", got, tt.want) }
    })
}
```

Идиоматичный способ тестирования в Go. Все кейсы в одном месте.

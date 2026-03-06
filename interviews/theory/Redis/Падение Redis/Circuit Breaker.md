Проблема: Redis лежит, но мы на каждый запрос ждём таймаут (3 сек) прежде чем пойти в БД. Сервис тормозит.
```
Без circuit breaker:
Запрос → Redis (ждём 3 сек таймаут) → БД → ответ через 3.05 сек

С circuit breaker:
Запрос → circuit открыт, Redis пропускаем → БД → ответ через 0.05 сек
```

Три состояния:
```
CLOSED  → всё ок, запросы идут в Redis
         ↓ N ошибок подряд
OPEN    → Redis пропускаем, сразу идём в БД
         ↓ через 30 сек пробуем один запрос
HALF-OPEN → если ответил — CLOSED, если нет — OPEN
```


```go
// Используем sony/gobreaker или самописный
breaker := gobreaker.NewCircuitBreaker(gobreaker.Settings{
    Name:        "redis",
    MaxRequests: 3,              // в HALF-OPEN пропускаем 3 запроса
    Interval:    10 * time.Second,
    Timeout:     30 * time.Second, // через 30 сек пробуем снова
    ReadyToTrip: func(counts gobreaker.Counts) bool {
        return counts.ConsecutiveFailures > 5 // 5 ошибок подряд → OPEN
    },
})

func GetFromCache(key string) (string, error) {
    result, err := breaker.Execute(func() (interface{}, error) {
        return redis.Get(ctx, key).Result()
    })
    if err != nil {
        return "", err // circuit open или Redis ошибка — идём в БД
    }
    return result.(string), nil
}
```

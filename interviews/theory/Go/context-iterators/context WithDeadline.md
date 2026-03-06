- автоотмена в указанный момент времени (абсолютное время, time.Time) ^deadline-absolute
- WithTimeout = обёртка над WithDeadline (duration → time.Now().Add(d)) ^deadline-wraps-timeout
- если у родителя дедлайн раньше — используется дедлайн родителя (ребёнок не может продлить время) ^deadline-parent-wins

---

## Когда WithDeadline вместо WithTimeout

В 99% случаев WithTimeout удобнее. ^deadline-vs-timeout-99

WithDeadline нужен для deadline propagation — когда пробрасываете конкретное время между сервисами: ^deadline-propagation-usecase

```go
// Клиент шлёт: "ответь до 15:00:00.000"
// Сервис A получает дедлайн, передаёт дальше сервису B
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(parent, deadline)
defer cancel()

// Передаём дедлайн в RPC-запрос к другому сервису
response, err := serviceB.Call(ctx, request)
```

^deadline-propagation-example

## Проверка дедлайна

```go
if deadline, ok := ctx.Deadline(); ok {
    remaining := time.Until(deadline)
    if remaining < 100*time.Millisecond {
        // слишком мало времени — не стоит начинать тяжёлую операцию
        return ErrNotEnoughTime
    }
}
```

^deadline-check-remaining

Паттерн позволяет не начинать тяжёлые операции если времени уже практически нет. ^deadline-check-purpose

## Связь
- [[context WithTimeout]] — относительное время (используется в 99% случаев)
- [[context WithCancel]] — без автоотмены по времени

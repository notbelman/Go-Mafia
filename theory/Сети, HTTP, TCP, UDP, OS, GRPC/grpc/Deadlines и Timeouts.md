- **Timeout** = "5 секунд от сейчас". **Deadline** = абсолютная точка во времени. gRPC конвертирует timeout→deadline, при передаче между сервисами **вычитает** прошедшее
- **Правило: каждый gRPC-вызов должен иметь deadline.** Без него — утечка горутин, исчерпание соединений, каскадный отказ
- **Propagation**: в Go deadline **автоматически** через context. Клиент 5s → Service A потратил 2s → Service B получит ~3s
- **Каскадные таймауты**: upstream > сумма downstream + retries. Одинаковый timeout везде = ошибка

---

## Propagation

В Go deadline **автоматически прокидывается** через контекст. Клиент поставил 5s → Service A потратил 2s → Service B получит ~3s. Если deadline истёк — все downstream отменяются через `ctx.Done()`.

## Каскадные таймауты

```
API Gateway: 10s
  └── Service A: 7s
        ├── DB: 3s (× 2 попытки)
        └── Cache: 500ms
```

**Правило:** upstream timeout > сумма downstream + retries. Ошибка — одинаковый timeout везде: низ ещё работает, верх уже отвалился.

## На практике

```go
// Клиент
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
resp, err := client.GetOrder(ctx, req)

// Сервер: проверять ctx перед дорогими операциями
if ctx.Err() != nil {
    return nil, status.Error(codes.DeadlineExceeded, "deadline exceeded")
}
result, err := db.Query(ctx, ...)
```

## Связь
- [[gRPC в Go — практика]] — context.WithTimeout на каждый вызов
- [[HTTP таймауты]] — аналогия: Timeout vs Deadline
- [[Interceptors]] — deadline прокидывается через interceptor chain

- **Interceptors** = middleware для gRPC. Код до/после каждого RPC. Задачи: logging, метрики, auth, tracing, panic recovery
- **4 типа**: Unary Server, Stream Server, Unary Client, Stream Client. Цепочка через `grpc.ChainUnaryInterceptor()`
- Порядок **важен**: metrics (оборачивает всё) → logging → auth (если не прошёл — handler не вызовется) → recovery (ловит panic)
- **go-grpc-middleware v2** — стандарт де-факто: готовые interceptors + `selector` (auth не для health check)

---

Аналог HTTP middleware для gRPC. Не привязан к конкретному методу — переиспользуется между сервисами.

## 4 типа

| | Unary | Stream |
|:--------|:------|:-------|
| **Server** | `UnaryServerInterceptor` | `StreamServerInterceptor` |
| **Client** | `UnaryClientInterceptor` | `StreamClientInterceptor` |

## Цепочка (chain)

```go
grpc.ChainUnaryInterceptor(
    metrics,    // первый: замеряет время всего вызова
    logging,    // второй
    auth,       // третий: если не прошёл — handler не вызовется
    recovery,   // последний: ловит panic из handler'а
)
```

## Пример: unary server interceptor

```go
func LoggingInterceptor(ctx context.Context, req any,
    info *grpc.UnaryServerInfo, handler grpc.UnaryHandler,
) (any, error) {
    start := time.Now()
    resp, err := handler(ctx, req) // вызов следующего interceptor'а или handler'а
    log.Printf("method=%s duration=%s err=%v",
        info.FullMethod, time.Since(start), err)
    return resp, err
}
```

Паттерн: pre-processing → `handler(ctx, req)` → post-processing.

## go-grpc-middleware v2

Библиотека готовых interceptors от grpc-ecosystem. Включает: logging, auth, recovery, prometheus, retry, rate limiting. `selector` — применять interceptor выборочно (health check без auth).

## Связь
- [[Auth через interceptor]] — конкретный пример: JWT в metadata
- [[gRPC в Go — практика]] — как подключить к серверу
- [[Deadlines и Timeouts]] — deadline прокидывается через ctx в interceptor chain

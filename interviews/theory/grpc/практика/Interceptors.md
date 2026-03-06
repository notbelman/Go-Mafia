Аналог HTTP middleware для gRPC. Код, который выполняется до/после
каждого RPC-вызова. Не привязан к конкретному методу — переиспользуется между сервисами.

Типичные задачи: логирование, метрики, auth, tracing, panic recovery.

## 4 типа

| | Unary | Stream |
|:--------|:------|:-------|
| **Server** | `UnaryServerInterceptor` | `StreamServerInterceptor` |
| **Client** | `UnaryClientInterceptor` | `StreamClientInterceptor` |

Unary — один запрос/ответ. Stream — перехватывает каждое сообщение в потоке.
Серверные — обрабатывают входящие вызовы. Клиентские — исходящие.

## Цепочка (chain)

К серверу или клиенту можно подключить несколько interceptors.
Выполняются последовательно, порядок важен:
```go
grpc.ChainUnaryInterceptor(
    metrics,    // первый: оборачивает всё, замеряет время всего вызова
    logging,    // второй
    auth,       // третий: если не прошёл — handler не вызовется
    recovery,   // последний: ловит panic из handler'а
)
```

## go-grpc-middleware v2

Библиотека готовых interceptors от grpc-ecosystem (стандарт де-факто).
Включает: logging, auth, recovery, prometheus, retry, rate limiting.
`selector` — позволяет применять interceptor выборочно по методу
(напр. health check без auth).

## Пример: unary server interceptor с нуля
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

Паттерн всегда один: pre-processing → `handler(ctx, req)` → post-processing.
Можно модифицировать ctx, req до вызова и resp, err после.
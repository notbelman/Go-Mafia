- Стандартный сервис `grpc.health.v1.Health`: `Check` (одноразовый) и `Watch` (стриминг). Статусы: `SERVING`, `NOT_SERVING`, `UNKNOWN`
- **Keepalive** = соединение живо? **Health check** = сервис **готов** обрабатывать запросы? Соединение может быть живым, но сервис не готов (нет БД, миграция)
- **K8s**: Liveness (жив? если нет → перезапуск) vs Readiness (готов? если нет → убрать из балансировки). Нативный gRPC probe с K8s 1.24+

---

## Протокол

`grpc.health.v1.Health` с двумя RPC:
- `Check` — одноразовый запрос статуса
- `Watch` — стриминг изменений статуса

Можно проверять как весь сервер (пустое имя), так и конкретный сервис.

## Liveness vs Readiness (Kubernetes)

- **Liveness** — процесс жив? Если нет → перезапустить pod
- **Readiness** — готов принимать трафик? Если нет → убрать из балансировки

Пример: сервис стартовал, ждёт прогрева кэша. Liveness = ok, Readiness = not serving.

## K8s (нативный gRPC probe, 1.24+)

```yaml
readinessProbe:
  grpc:
    port: 50051
livenessProbe:
  grpc:
    port: 50051
```

## На практике

```go
healthServer := health.NewServer()
grpc_health_v1.RegisterHealthServer(s, healthServer)
healthServer.SetServingStatus("myservice", grpc_health_v1.HealthCheckResponse_SERVING)
// При потере БД:
healthServer.SetServingStatus("myservice", grpc_health_v1.HealthCheckResponse_NOT_SERVING)
```

## Связь
- [[Keepalive]] — keepalive = соединение живо, health = сервис готов
- [[gRPC в Go — практика]] — регистрация health server

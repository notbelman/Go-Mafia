## Протокол
Стандартный сервис `grpc.health.v1.Health` с двумя RPC:
- `Check` — одноразовый запрос статуса
- `Watch` — стриминг изменений статуса

Статусы: `SERVING`, `NOT_SERVING`, `UNKNOWN`, `SERVICE_UNKNOWN`.
Можно проверять как весь сервер (пустое имя), так и конкретный сервис.

## Зачем (отличие от keepalive)
Keepalive — соединение живо? Health check — сервис готов обрабатывать запросы?
Соединение может быть живым, но сервис не готов (нет подключения к БД, миграция).

## Liveness vs Readiness (Kubernetes)
- **Liveness** — процесс жив? Если нет → перезапустить pod
- **Readiness** — готов принимать трафик? Если нет → убрать из балансировки

Пример: сервис стартовал, но ждёт прогрева кэша. Liveness = ok,
Readiness = not serving. Трафик не пойдёт пока кэш не прогреется.

## Интеграция с Kubernetes

**Нативный gRPC probe** (с K8s 1.24+):
```yaml
readinessProbe:
  grpc:
    port: 50051
livenessProbe:
  grpc:
    port: 50051
```

**Старый способ** — exec + `grpc_health_probe` бинарник в контейнере.

## На практике
В Go: `health.NewServer()` → `RegisterHealthServer()` → готово.
Менять статус: `SetServingStatus("myservice", NOT_SERVING)` —
например при потере коннекта к БД или при graceful shutdown.
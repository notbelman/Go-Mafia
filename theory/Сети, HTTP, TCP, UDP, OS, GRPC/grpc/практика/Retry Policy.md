## Два вида ретраев в gRPC

**Transparent retry** — автоматический, если данные ещё не отправлены
на сервер (ошибка соединения до передачи). Всегда включён, не настраивается.

**Policy-based retry** — настраиваемый, через service config (JSON).
Срабатывает когда сервер ответил retryable статус-кодом.

## Конфигурация

Задаётся как JSON в `grpc.WithDefaultServiceConfig()` на клиенте:
- `MaxAttempts` — макс попыток (включая первую), лимит 5 по умолчанию
- `InitialBackoff`, `MaxBackoff`, `BackoffMultiplier` — экспоненциальный backoff
- `RetryableStatusCodes` — на какие коды ретраить (обычно `UNAVAILABLE`)

Backoff: после каждой неудачи задержка × multiplier + jitter (±20%).
Deadline общий на все попытки — если истёк, ретраи прекращаются.

## Throttling (защита от перегрузки)

`retryThrottling`: клиент считает token_count (начинается с maxTokens).
Неудача −1, успех +tokenRatio. Если count < maxTokens/2 — ретраи
приостанавливаются. Защищает сервер от шторма ретраев.

## Что ретраить, а что нет

✅ `UNAVAILABLE` — сервис временно недоступен, смысл повторить.
⚠️ `DEADLINE_EXCEEDED`, `RESOURCE_EXHAUSTED` — осторожно, может усугубить.
❌ `INVALID_ARGUMENT`, `NOT_FOUND`, `ALREADY_EXISTS` — повтор бессмысленен.

Мутации — только с idempotency key, иначе дублирование данных.
- **Transparent retry** — автоматический, если данные **не отправлены** (ошибка соединения до передачи). Всегда включён
- **Policy-based retry** — настраиваемый через service config (JSON). `MaxAttempts`, exponential `Backoff` + jitter, `RetryableStatusCodes`
- **Throttling**: token_count. Неудача −1, успех +tokenRatio. count < maxTokens/2 → ретраи **приостанавливаются**. Защита от шторма
- ✅ Retry `UNAVAILABLE`. ❌ Не retry `INVALID_ARGUMENT`, `NOT_FOUND`. ⚠️ Мутации — только с **idempotency key**

---

## Два вида

**Transparent retry** — если данные ещё не отправлены. Всегда включён, не настраивается.

**Policy-based retry** — через `grpc.WithDefaultServiceConfig()`:
- `MaxAttempts` — макс попыток (включая первую), лимит 5
- `InitialBackoff`, `MaxBackoff`, `BackoffMultiplier` — экспоненциальный backoff
- `RetryableStatusCodes` — на какие коды ретраить

Backoff: задержка × multiplier + jitter (±20%). Deadline общий на все попытки.

## Throttling

`retryThrottling`: клиент считает token_count (начинается с maxTokens). Неудача −1, успех +tokenRatio. count < maxTokens/2 → ретраи приостанавливаются.

## Что ретраить

✅ `UNAVAILABLE` — временно недоступен.
⚠️ `DEADLINE_EXCEEDED`, `RESOURCE_EXHAUSTED` — осторожно.
❌ `INVALID_ARGUMENT`, `NOT_FOUND`, `ALREADY_EXISTS` — бессмысленно.

Мутации — только с idempotency key, иначе дублирование.

## Связь
- [[Deadlines и Timeouts]] — deadline общий на все попытки
- [[Идемпотентность HTTP]] — retry безопасен только для идемпотентных операций
- [[Status Codes и обработка ошибок]] — RetryInfo в details

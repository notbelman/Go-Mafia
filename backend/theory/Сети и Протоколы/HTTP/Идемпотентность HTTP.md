- POST/PATCH **не идемпотентны** по спеке. Retry = дубликат. 4 способа: **Idempotency-Key** (UUID в header), **hash payload**, **уникальный бизнес-ключ** (email), **ETag + If-Match**
- **Idempotency-Key** — самый распространённый: клиент генерит UUID, сервер кэширует response. Повторный запрос с тем же ключом → вернуть кэшированный ответ
- PATCH идемпотентен если **SET** (`{email: "x"}`), не идемпотентен если **INCREMENT** (`{balance: "+100"}`)

---

## 1. Idempotency-Key header

Клиент шлёт UUID в хедере, сервер кеширует response.
```
POST /orders
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

Retry с тем же ключом → вернёт закешированный response
```

## 2. Хэширование payload

Сервер вычисляет hash(request body), проверяет дубли.
```
POST /orders
{"item": "laptop", "price": 1000}

hash = SHA256(body) = "a3f8..."
Повторный POST с тем же телом → видит hash → возвращает старый response
```

## 3. Уникальный бизнес-ключ

```
POST /users
{"email": "user@example.com"}

Повторный POST → 409 Conflict (email уже есть в БД)
```

## 4. ETag + If-Match (для PATCH)

```
GET /users/123 → ETag: "v1"

PATCH /users/123
If-Match: "v1"
{"email": "new@mail.com"}

Повторный PATCH с тем же ETag → 412 Precondition Failed
```

## 5. Дизайн операций (для PATCH)

Идемпотентно: `{email: "new@mail.com"}` — замена значения.
НЕ идемпотентно: `{balance: "+100"}` — increment операция.

## Связь
- [[HTTP методы]] — safe/idempotent/cacheable свойства
- [[HTTP Status Codes]] — 409 Conflict, 412 Precondition Failed

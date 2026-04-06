- **Ресурсы = существительные** (`/users`, `/orders`), не глаголы (`/getUsers`). Множественное число. Вложенность: `/users/123/orders` (заказы юзера)
- **Versioning**: URI-based (`/v1/users`) — самый распространённый. Header-based (`Accept-Version: 1`) — чище, но сложнее. Не ломай клиентов — **добавляй** поля, не удаляй
- **Pagination**: offset (`?page=2&limit=20` — просто, плохо на больших offset'ах) vs **cursor** (`?after=eyJpZCI6MTIzfQ` — стабильно, O(1)). Cursor для прода
- **Error format**: стандартизируй структуру ошибок. `{"error": {"code": "USER_NOT_FOUND", "message": "...", "details": [...]}}`
- Антипаттерны: глаголы в URL, один endpoint на всё (POST /api), игнорирование статус-кодов (всё 200 + `{"success": false}`)

---

## Naming

```
✅ ХОРОШО                         ❌ ПЛОХО
GET    /users                     GET    /getUsers
GET    /users/123                 GET    /getUserById?id=123
POST   /users                     POST   /createUser
DELETE /users/123                 POST   /deleteUser
GET    /users/123/orders          GET    /getUserOrders?userId=123
POST   /users/123/orders          POST   /createOrderForUser
```

**Правила:**
- Существительные, не глаголы (метод = глагол)
- Множественное число (`/users`, не `/user`)
- Kebab-case (`/order-items`, не `/orderItems`)
- Вложенность ≤ 2 уровня (`/users/123/orders` ОК, `/users/123/orders/456/items/789` — слишком глубоко)

### Actions (RPC-style в REST)

Некоторые операции не ложатся на CRUD:

```
POST /orders/123/cancel          ← действие
POST /users/123/reset-password   ← действие
POST /reports/generate           ← запуск процесса
```

Это нормально. REST — не догма. Действие = POST на подресурс.

## Versioning

| Подход | Пример | Плюсы | Минусы |
|:--|:--|:--|:--|
| **URI** | `/v1/users` | Просто, очевидно, кэшируется | URL меняется |
| **Header** | `Accept-Version: 1` | URL не меняется | Сложнее тестировать (curl) |
| **Query** | `/users?version=1` | Просто | Ломает кэширование |

**На практике**: URI (`/v1/`, `/v2/`) — стандарт индустрии. Stripe, GitHub, Twilio — все так делают.

**Золотое правило**: **не ломай клиентов**. Добавлять поля в ответ — ОК (backward compatible). Удалять/переименовывать — breaking change → новая версия.

## Pagination

### Offset-based

```
GET /users?page=2&limit=20
→ {"data": [...], "total": 1000, "page": 2, "limit": 20}
```

**Проблема**: `page=5000` → БД делает `OFFSET 100000` → сканирует и пропускает 100K строк → медленно.

### Cursor-based

```
GET /users?limit=20
→ {"data": [...], "next_cursor": "eyJpZCI6MTIzfQ"}

GET /users?limit=20&after=eyJpZCI6MTIzfQ
→ {"data": [...], "next_cursor": "eyJpZCI6MTQzfQ"}
```

Cursor = base64(последний id/timestamp). Внутри: `WHERE id > 123 LIMIT 20` → **O(1)** через индекс, не O(N) через OFFSET.

**Минус cursor**: нельзя прыгнуть на "страницу 50" (только вперёд/назад).

## Фильтрация и сортировка

```
GET /users?status=active&role=admin           ← фильтрация
GET /users?sort=-created_at,name              ← сортировка (- = DESC)
GET /users?fields=id,name,email               ← выбор полей (partial response)
GET /orders?created_at[gte]=2024-01-01        ← range фильтр
```

## Error Format

```json
// ❌ ПЛОХО: непонятно что пошло не так
{"error": "Something went wrong"}

// ✅ ХОРОШО: машиночитаемый код + человекочитаемое сообщение
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {"field": "email", "message": "Invalid email format"},
      {"field": "age", "message": "Must be positive"}
    ]
  }
}
```

**Стандарты**: RFC 7807 (Problem Details for HTTP APIs):
```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Error",
  "status": 422,
  "detail": "Email format is invalid",
  "instance": "/users/123"
}
```

## Антипаттерны

```
❌ POST /api {"action": "getUser", "id": 123}    ← Level 0, RPC не REST
❌ Всё 200 OK + {"success": false, "error": "..."} ← игнорирование статус-кодов
❌ GET /getUsers, POST /createUser                  ← глаголы в URL
❌ PUT /users/123 {"name": "Ivan"}                  ← PUT без полного объекта (это PATCH)
❌ DELETE /users/123 → 200 {"deleted": true}        ← 204 No Content правильнее
```

## Связь
- [[REST — архитектурный стиль]] — теория: constraints, Richardson levels
- [[REST — HTTP версии и ограничения]] — когда REST не подходит
- [[HTTP методы]] — CRUD → GET/POST/PUT/PATCH/DELETE
- [[HTTP Status Codes]] — правильные коды для каждой операции

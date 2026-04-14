- **REST** = HTTP методы + ресурсы (URL) + статус-коды + stateless. **Level 2** (Richardson) = стандарт индустрии. Level 3 (HATEOAS) = теория, почти никто не делает
- **gRPC** = бинарный (Protobuf) + HTTP/2 + streaming + code generation. Для **микросервисов** и high-load (internal API)
- **GraphQL** = клиент выбирает поля. Один endpoint `POST /graphql`. Для сложных UI, мобильных приложений (экономия трафика)
- Правило: **REST** = default. **gRPC** = internal service-to-service. **GraphQL** = сложный frontend с разными вьюшками

---

## Сравнение

| | REST | gRPC | GraphQL | JSON-RPC |
|:--|:--|:--|:--|:--|
| Формат | JSON | **Protobuf** (binary) | JSON | JSON |
| Транспорт | HTTP/1.1 | **HTTP/2** | HTTP/1.1 | HTTP/1.1 |
| Типизация | Нет (OpenAPI) | **.proto** (строгая) | **Schema** (строгая) | Нет |
| Streaming | Нет (WebSocket) | **Да** (4 вида) | Subscriptions | Нет |
| Кэширование | **Да** (GET кэшируется) | Сложно | Сложно (всё POST) | Нет |
| Когда | CRUD, публичные API | Микросервисы, high-load | Сложный UI, mobile | Простые internal API |

## REST

```
GET    /users/123     → 200 OK {"id": 123, "name": "Ivan"}
POST   /users         → 201 Created + Location: /users/123
PUT    /users/123     → 200 OK
PATCH  /users/123     → 200 OK
DELETE /users/123     → 204 No Content
```

**Плюсы**: простота, кэширование (GET), stateless, HTTP семантика.
**Минусы**: over-fetching (получаешь всё, нужно только name), under-fetching (user + orders = 2 запроса), нет строгой типизации.

## Richardson Maturity Model

```
Level 0: один URL, один POST, action в теле (RPC, не REST)
         POST /api {"action": "getUser", "id": 123}

Level 1: ресурсы (разные URL), но всё POST
         POST /users/123

Level 2: правильные методы + статус-коды     ← ЭТО НАЗЫВАЮТ "REST"
         GET /users/123 → 200 OK

Level 3: HATEOAS — сервер возвращает ссылки
         {"_links": {"orders": "/users/123/orders"}}  ← теория
```

**На собесе "RESTful API" = Level 2**. Level 3 спрашивают чтобы проверить знает ли кандидат что "настоящий REST" включает HATEOAS.

## gRPC

```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListRequest) returns (stream User);  // server streaming
}
```

```go
user, err := client.GetUser(ctx, &pb.GetUserRequest{Id: 123})
```

**4 вида streaming**: unary, server-stream, client-stream, bidirectional.

**Плюсы**: быстрый (binary), строгая типизация, code generation, streaming.
**Минусы**: не читается человеком, сложнее дебажить, нужен gRPC-Web для браузеров.

## GraphQL

```graphql
POST /graphql
{
  user(id: 123) {
    name
    orders { id, total }
  }
}
```

**Один запрос** вместо GET /users/123 + GET /users/123/orders.
**Только нужные поля** — нет over-fetching.
**N+1 проблема** — нужен DataLoader.

## Связь
- [[HTTP методы]] — REST = методы + ресурсы
- [[HTTP протокол и версии]] — gRPC требует HTTP/2
- [[WebSocket]] — real-time альтернатива REST polling

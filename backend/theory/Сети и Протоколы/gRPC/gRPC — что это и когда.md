- **gRPC** = HTTP/2 + Protobuf + кодогенерация. Вызов функций на удалённом сервере **как локально**. Google, 2015
- **Почему быстрее REST**: бинарный Protobuf (в 5-10x компактнее JSON), HTTP/2 мультиплексинг (один TCP на всё), header compression (HPACK), persistent connection (нет handshake на каждый запрос)
- **REST** = ресурсы + CRUD (существительные: `/users/123`). **gRPC** = процедуры (глаголы: `GetUser(id)`, `TransferMoney(from, to, amount)`)
- **Когда REST**: публичный API, браузер, webhooks, простой CRUD. **Когда gRPC**: межсервисное (internal), high RPS, streaming, строгий контракт (.proto = source of truth)
- **gRPC-Gateway**: один сервис, два протокола. Наружу REST (JSON), внутри gRPC (Protobuf)

---

## Как работает

```
gRPC = HTTP/2 + Protobuf + кодогенерация
         │          │            │
     транспорт  сериализация  клиент/сервер из .proto
```

```
Client                          Server
  │  CreateUser(request)          │
  │  1. Protobuf encode ──────►   │  2. Protobuf decode
  │                               │  3. Выполнить логику
  │  5. Protobuf decode  ◄──────  │  4. Protobuf encode
  │  response                     │
```

Клиент вызывает метод как **локальную функцию**. Под капотом: сериализация → HTTP/2 → десериализация.

## Почему gRPC быстрее REST

| Фактор | REST (JSON/HTTP1.1) | gRPC (Protobuf/HTTP2) |
|:--|:--|:--|
| **Сериализация** | JSON: текст, парсинг строк | Protobuf: бинарный, **5-10x компактнее** |
| **Транспорт** | HTTP/1.1: новое соединение или keep-alive очередь | HTTP/2: **мультиплексинг**, один TCP |
| **Headers** | Повторяются в каждом запросе (Cookie, Auth, Content-Type) | **HPACK**: сжатие, дедупликация headers |
| **Соединение** | Может переустанавливаться | **Persistent**: одно соединение, все запросы |
| **Типизация** | Runtime (JSON → `interface{}` → type assertion) | **Compile-time** (кодогенерация, zero allocation decode) |

**Бенчмарк** (типичный): gRPC на **2-10x** быстрее REST по latency на internal вызовах. На маленьких payload'ах разница меньше, на больших — значительнее.

## REST vs gRPC

|           | REST                             | gRPC                            |
|:----------|:---------------------------------|:--------------------------------|
| Модель    | Ресурсы + CRUD (существительные) | Вызов процедур (глаголы)        |
| Формат    | JSON (текст, человекочитаемый)   | Protobuf (бинарный, компактный) |
| Контракт  | OpenAPI/Swagger (опционально)    | .proto (обязательно)            |
| Транспорт | HTTP/1.1 или HTTP/2              | Только HTTP/2                   |
| Стриминг  | Нет (WebSocket отдельно)         | Встроенный, в обе стороны       |
| Браузер   | Нативно                          | Нужен gRPC-Web или прокси       |
| Отладка   | curl, Postman                    | grpcurl, Evans, Postman (новые) |

```
REST — вокруг ресурсов:       gRPC — вокруг действий:
  GET    /users/123              GetUser(id)
  POST   /users                  CreateUser(data)
  DELETE /users/123              TransferMoney(from, to, amount)
```

## Когда REST

- Публичный API для внешних клиентов (все умеют HTTP+JSON)
- Браузерные приложения без прокси
- Webhooks (принимающая сторона ожидает HTTP)
- Простые CRUD-интеграции с третьими сторонами

## Когда gRPC

- **Межсервисное** общение (внутри кластера)
- Высокий RPS, критична латенси
- **Стриминг** (real-time обновления, чат, логи)
- Строгий контракт между командами (.proto = source of truth)
- **Полиглот** (один proto → Go, Python, Java клиенты)

## gRPC-Gateway — оба сразу

```
Браузер → REST (JSON) → gRPC-Gateway → gRPC (Protobuf) → Сервис
```

Маршруты в proto:
```protobuf
import "google/api/annotations.proto";

rpc GetOrder(GetOrderRequest) returns (Order) {
    option (google.api.http) = { get: "/v1/orders/{id}" };
}
```

Один сервис, один handler, два протокола.

**ConnectRPC** (Buf) — альтернатива: совместим с gRPC, но работает поверх HTTP/1.1+JSON без прокси. Браузер вызывает напрямую.

## Можно ли gRPC в обычном HTTP?

- **gRPC-Web**: подмножество gRPC для браузеров, через прокси (Envoy)
- **ConnectRPC**: gRPC-совместимый, работает поверх HTTP/1.1 напрямую
- **REST поверх gRPC**: gRPC-Gateway генерирует REST reverse proxy
- Технически Protobuf можно использовать с любым транспортом, но gRPC = HTTP/2

## Связь
- [[Protobuf и proto файлы]] — формат данных и контракт
- [[Типы gRPC вызовов]] — unary, server/client/bidi streaming
- [[REST — HTTP версии и ограничения]] — ограничения REST, которые gRPC решает
- [[HTTP протокол и версии]] — HTTP/2 мультиплексинг

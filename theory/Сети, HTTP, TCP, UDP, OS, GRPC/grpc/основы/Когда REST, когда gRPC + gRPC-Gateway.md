## Когда REST
- Публичный API для внешних клиентов (все умеют HTTP+JSON)
- Браузерные приложения без прокси
- Webhooks (принимающая сторона ожидает HTTP)
- Простые CRUD-интеграции с третьими сторонами

## Когда gRPC
- Межсервисное общение (внутри кластера)
- Высокий RPS, критична латенси (бинарный формат, HTTP/2)
- Стриминг (real-time обновления, чат, логи)
- Строгий контракт между командами (proto = source of truth)
- Полиглот (один proto → Go, Python, Java клиенты)

## Когда оба — gRPC-Gateway

Ситуация: внутри gRPC, но нужен REST наружу.
gRPC-Gateway — protoc-плагин, генерирует reverse proxy:
```
Внешний клиент → REST (JSON) → gRPC-Gateway → gRPC (Protobuf) → Сервис
```

Маршруты описываются прямо в proto:
```protobuf
import "google/api/annotations.proto";

rpc GetOrder(GetOrderRequest) returns (Order) {
    option (google.api.http) = {
        get: "/v1/orders/{id}"
    };
}
```

Один сервис, один handler, два протокола. REST-клиенты получают JSON,
внутренние сервисы работают через gRPC напрямую.

## Альтернатива — ConnectRPC
Протокол от Buf. Совместим с gRPC, но работает поверх HTTP/1.1+JSON
без прокси. Браузер вызывает напрямую. Набирает популярность,
но экосистема меньше чем у gRPC.
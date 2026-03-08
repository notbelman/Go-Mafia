- **Protobuf** — бинарный формат сериализации Google. Данные = пары `номер_поля + значение`. **Имя поля НЕ передаётся** по сети — только номер и тип
- **package** = полное имя сервиса (`order.v1.OrderService`). Сменил package → клиент не найдёт сервис. Одинаковые типы и номера, но разные имена → сериализация работает, **роутинг нет**
- В Go: `message` → struct, `enum` → int32+константы, `repeated` → slice, `optional` → pointer (*T), `map` → map, `service` → interface + клиент
- Альтернативы Protobuf: **FlatBuffers** (zero-copy, игры), **MessagePack** (бинарный JSON), **Avro** (schema в рантайме, Kafka), **Thrift** (Facebook). gRPC технически может работать с JSON, но **теряет** главное преимущество — скорость

---

## Пример proto файла

```protobuf
syntax = "proto3";
package order.v1;
option go_package = "gen/order/v1";

enum OrderStatus {
    ORDER_STATUS_UNSPECIFIED = 0;  // всегда нулевой default
    ORDER_STATUS_PENDING = 1;
    ORDER_STATUS_COMPLETED = 2;
}

message Order {
    int64 id = 1;
    string user_id = 2;
    repeated Item items = 3;       // slice в Go
    OrderStatus status = 4;
    optional string comment = 5;   // *string в Go (отличить "не передали" от "")
    map<string, string> meta = 6;  // map[string]string в Go
}

message Item {
    string name = 1;
    int32 quantity = 2;
    double price = 3;
}

service OrderService {
    rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
    rpc ListOrders(ListOrdersRequest) returns (stream Order);
}
```

## Маппинг в Go

| Protobuf | Go | Нюанс |
|:--|:--|:--|
| `message` | struct | + Marshal/Unmarshal |
| `enum` | int32 + константы | 0 = UNSPECIFIED (всегда) |
| `repeated T` | `[]T` | nil если не передали |
| `optional T` | `*T` (pointer) | nil vs zero value |
| `map<K,V>` | `map[K]V` | nil если не передали |
| `service` | interface (server) + struct (client) | кодогенерация |
| `bytes` | `[]byte` | |
| `google.protobuf.Timestamp` | `timestamppb.Timestamp` | `.AsTime()` → `time.Time` |

## Package — зачем важен

```protobuf
package order.v1;
```

gRPC роутит вызовы по **полному имени**: `/order.v1.OrderService/CreateOrder`.

- Сменил package → клиент шлёт запрос на старое имя → `UNIMPLEMENTED`
- Одинаковые типы и номера полей, **разные package** → Protobuf десериализует (бинарно совпадает), но gRPC **не найдёт** сервис

## optional vs default

Proto3: все поля имеют **default zero value**. `int32 age = 3` → если не передали, age = 0. Нельзя отличить "передали 0" от "не передали".

**optional** решает: `optional int32 age = 3` → в Go `*int32`. `nil` = не передали, `new(int32)` с 0 = передали ноль.

## Альтернативы Protobuf

| Формат | Особенность | Когда |
|:--|:--|:--|
| **Protobuf** | Компактный бинарный, schema required | gRPC, high-load |
| **FlatBuffers** | Zero-copy (доступ без десериализации) | Игры, real-time |
| **MessagePack** | Бинарный JSON (schema optional) | Когда JSON слишком большой |
| **Avro** | Schema в рантайме, schema evolution | Kafka, BigData |
| **Thrift** | Protobuf-аналог от Facebook | Legacy Facebook-экосистема |
| **JSON** | Текстовый, человекочитаемый | REST, дебаг |

**Можно ли gRPC с JSON?** Технически да (`grpc-json` codec). Но теряешь: компактность, скорость encode/decode, строгую типизацию. Смысл gRPC = Protobuf.

## Связь
- [[gRPC — что это и когда]] — gRPC = HTTP/2 + Protobuf + codegen
- [[Номера полей и совместимость]] — правила эволюции schema
- [[Кодогенерация и управление протофайлами]] — protoc vs buf, CI
- [[Валидация в gRPC]] — Protobuf гарантирует типы, но не семантику

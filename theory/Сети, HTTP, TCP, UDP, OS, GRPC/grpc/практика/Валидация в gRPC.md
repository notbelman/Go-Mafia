## Проблема
Protobuf гарантирует типы, но не семантику. Поле `string email`
примет любую строку. Нужна валидация на уровне контракта.

## protovalidate (от Buf, замена protoc-gen-validate)

Правила задаются прямо в `.proto` через аннотации:
```protobuf
import "buf/validate/validate.proto";

message CreateUserRequest {
    string email = 1 [(buf.validate.field).string.email = true];
    string name  = 2 [(buf.validate.field).string = {
        min_len: 1, max_len: 64
    }];
    uint32 age   = 3 [(buf.validate.field).uint32.lte = 150];
    string id    = 4 [(buf.validate.field).string.uuid = true];
}
```

Кастомные правила через CEL-выражения (на уровне message):
```protobuf
option (buf.validate.message).cel = {
    id: "password_match"
    message: "passwords must match"
    expression: "this.password == this.confirm_password"
};
```

## Применение через interceptor

Сами по себе аннотации **ничего не делают** — нужен interceptor:
```go
validator, _ := protovalidate.New()
grpc.NewServer(
    grpc.ChainUnaryInterceptor(
        protovalidate_middleware.UnaryServerInterceptor(validator),
    ),
)
```

При нарушении автоматически возвращается `InvalidArgument` с описанием
какое поле и какое правило нарушено (структурированный ответ).

## Зачем на уровне proto, а не в Go-коде

Валидация в `.proto` — часть контракта. Видна всем клиентам на любом языке.
Не дублируется между сервисами. Линтится через `buf lint`.
Ручная валидация в handler'е — для бизнес-правил, которые зависят от состояния
(напр. "баланс >= сумма перевода").
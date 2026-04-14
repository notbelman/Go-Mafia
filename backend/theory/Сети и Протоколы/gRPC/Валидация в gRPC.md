- **Protobuf гарантирует типы, но не семантику**. `string email` примет любую строку → нужна валидация
- **protovalidate** (Buf): правила прямо в `.proto` через аннотации. Кастомные правила через **CEL-выражения**. Применяется через **interceptor**
- Валидация в proto = **часть контракта**, видна всем клиентам на любом языке. Ручная валидация в handler — для **бизнес-правил** (баланс >= сумма)

---

## protovalidate (от Buf)

```protobuf
import "buf/validate/validate.proto";

message CreateUserRequest {
    string email = 1 [(buf.validate.field).string.email = true];
    string name  = 2 [(buf.validate.field).string = { min_len: 1, max_len: 64 }];
    uint32 age   = 3 [(buf.validate.field).uint32.lte = 150];
    string id    = 4 [(buf.validate.field).string.uuid = true];
}
```

Кастомные через CEL:
```protobuf
option (buf.validate.message).cel = {
    id: "password_match"
    message: "passwords must match"
    expression: "this.password == this.confirm_password"
};
```

## Interceptor

```go
validator, _ := protovalidate.New()
grpc.NewServer(
    grpc.ChainUnaryInterceptor(
        protovalidate_middleware.UnaryServerInterceptor(validator),
    ),
)
```

При нарушении → `InvalidArgument` с описанием поля и правила.

## Связь
- [[Protobuf и proto файлы]] — protobuf = типы, protovalidate = семантика
- [[Interceptors]] — валидация как interceptor
- [[Status Codes и обработка ошибок]] — InvalidArgument при нарушении

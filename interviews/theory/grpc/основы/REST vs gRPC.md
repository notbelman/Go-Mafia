
|           | REST                             | gRPC                            |
| :-------- | :------------------------------- | :------------------------------ |
| Модель    | Ресурсы + CRUD (существительные) | Вызов процедур (глаголы)        |
| Формат    | JSON (текст, человекочитаемый)   | Protobuf (бинарный, компактный) |
| Контракт  | OpenAPI/Swagger (опционально)    | .proto (обязательно)            |
| Транспорт | HTTP/1.1 или HTTP/2              | Только HTTP/2                   |
| Стриминг  | Нет (WebSocket отдельно)         | Встроенный, в обе стороны       |
| Браузер   | Нативно                          | Нужен gRPC-Web или прокси       |
| Отладка   | curl, Postman                    | grpcurl, Evans                  |

**Архитектурно:**
```
REST — вокруг ресурсов:       gRPC — вокруг действий:
  GET    /users/123              GetUser(id)
  POST   /users                  CreateUser(data)
  DELETE /users/123              TransferMoney(from, to, amount)
```

**Совмещение — gRPC-Gateway:**
```
Браузер → REST (JSON) → gRPC-Gateway → gRPC (Protobuf) → Сервис
```
Один сервис, два протокола. Наружу REST, внутри gRPC.
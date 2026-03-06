**gRPC (Google Remote Procedure Call)** — фреймворк для вызова функций на удалённом сервере так, как будто вызываешь локально.
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
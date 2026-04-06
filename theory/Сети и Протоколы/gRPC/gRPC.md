## Основы

- [[gRPC — что это и когда]] — что такое gRPC, когда использовать vs REST
- [[Типы gRPC вызовов]] — unary, server/client/bidirectional streaming
- [[Protobuf и proto файлы]] — синтаксис .proto, типы данных, структура
- [[Номера полей и совместимость]] — backward/forward compatibility, reserved fields
- [[Кодогенерация и управление протофайлами]] — protoc, плагины, buf

## Надёжность

- [[Status Codes и обработка ошибок]] — gRPC status codes, детали ошибок
- [[Deadlines и Timeouts]] — propagation, cancellation
- [[Retry Policy]] — retry policy, hedging
- [[Health Checking]] — gRPC health protocol

## Производительность и соединения

- [[Keepalive]] — keepalive параметры, настройка
- [[Load Balancing]] — client-side LB, service mesh, DNS

## Безопасность и расширяемость

- [[Interceptors]] — unary и stream interceptors, цепочки
- [[Auth через interceptor]] — JWT, mTLS через interceptors
- [[Валидация в gRPC]] — protoc-gen-validate, ручная валидация

## Практика

- [[gRPC в Go — практика]] — сервер, клиент, рефлексия, тестирование в Go

Одно сообщение -> ВСЕ подписчики получают копию (broadcast).

## Схема
```
Producer: UserRegistered event
  ↓
[Topic/Exchange]
  ↓ (копия)     ↓ (копия)        ↓ (копия)
Consumer 1:   Consumer 2:      Consumer 3:
billing       analytics        notifications
(отправить    (записать в      (отправить
счет)         ClickHouse)      welcome email)
```

Каждый консьюмер получает СВОЮ копию сообщения.

## RabbitMQ (Exchange Fanout)
Producer:
channel.Publish("user-events", msg)  // отправка в Exchange

Exchange "user-events" (type: fanout):
  -> Queue "billing"       -> Consumer 1
  -> Queue "analytics"     -> Consumer 2
  -> Queue "notifications" -> Consumer 3

Fanout = раздать всем очередям (broadcast).

## Kafka (Topic) (Дефолт)
Topic "user-events":
Partition 0: [msg1, msg2, msg3]

Consumer Group "billing":
  Consumer 1 -> читает Partition 0

Consumer Group "analytics":
  Consumer 2 -> читает Partition 0 (ту же самую)

Consumer Group "notifications":
  Consumer 3 -> читает Partition 0 (ту же самую)

Разные Consumer Groups читают ОДНИ И ТЕ ЖЕ сообщения (копии).
## Юзкейсы Pub/Sub
- Event-Driven архитектура (UserCreated, OrderPlaced)
- Микросервисы слушают события друг друга
- Логирование (все события идут в Elasticsearch)
- Кеш инвалидация (ProductUpdated -> сбросить кеш)
- Аудит (все события пишутся в audit log)

Продюсер НЕ знает сколько сервисов слушают топик (loose coupling).
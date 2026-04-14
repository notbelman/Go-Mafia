## RabbitMQ

### Архитектурные компоненты

#### [[Exchanges (Обменники)]]
Типы exchange: direct, fanout, topic, headers — как маршрутизируются сообщения

#### [[Queues (Очереди)]]
Очереди, durable vs transient, exclusive, auto-delete, аргументы (TTL, max-length)

#### [[Bindings (Привязки)]]
Binding key, routing key, связь exchange → queue

#### [[Как вся эту хуйня работает]]
Полный путь сообщения: producer → exchange → binding → queue → consumer

---

### Гарантии доставки

#### [[Механизм подтверждений (ACKs)]]
Consumer ACK/NACK/reject, publisher confirms, mandatory flag

#### [[Уровни гарантий доставки]]
At-most-once, at-least-once, persistent messages, durable queues

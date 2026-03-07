Одно сообщение обрабатывается ТОЛЬКО одним консьюмером.

## Схема
```
Producer: order1, order2, order3 -> [Queue] 
[Queue]: [order1] [order2] [order3] 
			↓       ↓       ↓ 
Worker 1: взял order1 (обрабатывает) 
Worker 2: взял order2 (обрабатывает) 
Worker 3: ждет 
Worker 4-10: ждут
```

После того как Worker 1 взял order1 -> order1 УХОДИТ из очереди (или блокируется до ACK).

## RabbitMQ (Дефолт)
Producer: channel.Publish("orders", msg)
Consumer: channel.Consume("orders", func(msg) { process(msg); msg.Ack() })

После Ack() сообщение удаляется из очереди навсегда.
Если Worker упал до Ack() -> сообщение вернется в очередь.

## Kafka (Consumer Groups)
Topic "orders" имеет 3 партиции:
Partition 0: [msg1, msg4] <- Consumer 1 читает
Partition 1: [msg2, msg5] <- Consumer 2 читает
Partition 2: [msg3, msg6] <- Consumer 3 читает

Одну партицию читает ТОЛЬКО один консьюмер в группе.
Сообщения НЕ удаляются, двигается offset (указатель).

## Юзкейсы
Обработка заказов, отправка email, генерация отчетов, background jobs.
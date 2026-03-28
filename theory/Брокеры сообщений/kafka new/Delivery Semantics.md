- **At-most-once:** сообщение доставлено 0 или 1 раз. Возможна потеря, дублей нет. `acks=0` или commit offset до обработки
- **At-least-once:** сообщение доставлено 1+ раз. Потери нет, но возможны дубли. `acks=all` + retries — **default Kafka**
- **Exactly-once:** сообщение обработано ровно 1 раз. Два механизма: **idempotent producer** (дедупликация retry) + **transactions** (atomic consume-process-produce)
- Exactly-once в Kafka работает **только** для паттерна consume→process→produce (stream processing). Для side effects (email, REST, DB) — не работает

---

## Три семантики доставки

```
                    Потеря?    Дубли?     Как достичь в Kafka
At-most-once        Да         Нет        acks=0, или commit offset до обработки
At-least-once       Нет        Да         acks=all + retries (default)
Exactly-once        Нет        Нет        idempotent producer + transactions
```
^ds-comparison

## At-most-once

```
Producer:  acks=0, fire-and-forget → если брокер не получил, сообщение потеряно
Consumer:  commit offset → потом обработка → крашнулся → offset committed,
           но данные не обработаны → потеряны
```
^ds-at-most-once

Используется когда потеря допустима: метрики, логи, клики. ^ds-at-most-once-usecase

## At-least-once

```
Producer:  acks=all + retries → если неясно, дошло или нет → retry
           → broker мог получить оба → дубль
Consumer:  обработка → commit offset → крашнулся между обработкой и commit
           → rebalance → новый consumer перечитает → дубль
```
^ds-at-least-once

**Default поведение Kafka.** Retries гарантируют доставку, но дубли возможны на обоих концах. ^ds-at-least-once-default

## Exactly-once

Два механизма, решающие разные проблемы: ^ds-exactly-once-mechanisms

```
1. Idempotent Producer (enable.idempotence=true)
   → решает: дубли от retry на стороне producer
   → НЕ решает: дубли от двух разных producer instances,
     дубли при reprocessing на consumer

2. Transactions (transactional.id + consume-process-produce)
   → решает: atomic multipartition write
     (offset commit + result produce = одна транзакция)
   → решает: zombie fencing (старый producer не может писать)
   → НЕ решает: side effects (email, REST API, DB write)
```

## Где exactly-once НЕ работает

```
✗ Side effects: отправка email, вызов REST API, запись в файл
✗ Kafka → Database: нет общей транзакции (используй outbox pattern)
✗ Database → Kafka → Database: consumer не знает границ транзакций
✗ Cross-cluster copy: MirrorMaker 2.0 копирует записи, но теряет транзакционность
✗ Pub/sub: consumer может обработать дважды (зависит от offset commit)
```
^ds-exactly-once-limitations

**Outbox pattern:** микросервис пишет в Kafka topic → relay service читает и обновляет DB идемпотентно. Или наоборот: пишет в DB outbox table → Debezium/CDC → Kafka. ^ds-outbox-pattern

## Связь
- [[Idempotent Producer]] — дедупликация retry через PID + sequence number
- [[Transactions]] — atomic consume-process-produce, zombie fencing
- [[Producer — надёжность]] — acks, retries (Ch.7)
- [[Consumer — надёжность]] — offset commit, retry patterns (Ch.7)
- [[Reliable Data Delivery — обзор]] — гарантии Kafka (Ch.7)
- [[Kafka Streams — архитектура]] — exactly-once через processing.guarantee (Ch.14)

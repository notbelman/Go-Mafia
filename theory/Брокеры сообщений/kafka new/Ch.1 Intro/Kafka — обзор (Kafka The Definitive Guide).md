- Kafka = **распределённый commit log** (distributed streaming platform). Pub/sub + дисковое хранение + горизонтальное масштабирование
- Данные организованы в **topics → partitions**. Partition = append-only log с гарантией порядка. Topic = логическая группировка
- **Producer** пишет в partition (через key → hash → partition). **Consumer** читает по offset. **Consumer group** — параллельное потребление
- **Broker** = один сервер Kafka. Кластер брокеров, один из них — **controller** (leader election). Partition реплицируется на несколько брокеров (leader + followers)
- **Retention:** время (7 дней default) или размер (1GB). Отдельно: **log compaction** — хранить последнее значение по ключу. Данные не удаляются после чтения (в отличие от очередей)

---

## Kafka vs традиционные очереди

```
Очередь (RabbitMQ, SQS):          Kafka:
- сообщение удаляется после ACK   - сообщение хранится (retention)
- один consumer на сообщение       - множество consumer groups
- нет replay                       - replay с любого offset
- push модель                      - pull модель (consumer.poll())
```
^ko-vs-queues

## Ключевые концепции

```
Message  — массив байт (key + value + headers + metadata)
Batch    — группа messages в одну partition (эффективность сети)
Topic    — логическая категория сообщений (≈ таблица в БД)
Partition — append-only log внутри topic. Порядок гарантирован ВНУТРИ partition
Offset   — уникальный ID сообщения в partition (монотонно растёт)
Schema   — формат данных (Avro/Protobuf + Schema Registry рекомендуется)
```
^ko-concepts

## Архитектура кластера

```
                    Kafka Cluster
    ┌──────────┬──────────┬──────────┐
    │ Broker 0 │ Broker 1 │ Broker 2 │
    │ (ctrl)   │          │          │
    └──────────┴──────────┴──────────┘
         │           │          │
    ┌────┴────┐ ┌────┴────┐ ┌──┴──────┐
    │ P0 lead │ │ P1 lead │ │ P2 lead │
    │ P1 foll │ │ P2 foll │ │ P0 foll │
    └─────────┘ └─────────┘ └─────────┘

Controller — один из брокеров, выбирает лидеров партиций
Leader — принимает produce/fetch. Followers — реплицируют
Producer → всегда к leader. Consumer → leader или follower (KIP-392)
```
^ko-cluster

## Почему Kafka

- **Multiple producers/consumers:** одна точка для всех данных, без координации
- **Disk-based retention:** consumer может быть offline, данные не теряются
- **Scalable:** от 1 брокера до сотен, расширение без downtime
- **High performance:** миллионы msg/sec, sub-second latency
- **Replay:** можно перечитать данные с любого offset
^ko-why-kafka

## Retention

```
Два типа:
  Time-based:  log.retention.hours=168 (7 дней default)
  Size-based:  log.retention.bytes=1073741824 (1GB)

Третий вариант:
  Log compaction: хранить последнее значение для каждого key
  → changelog, state store, CDC

Retention настраивается per-topic
```
^ko-retention

## Use Cases

- **Activity tracking:** клики, page views, действия пользователей (оригинальный use case LinkedIn)
- **Messaging:** уведомления, email (decouple отправителя от получателя)
- **Metrics & Logging:** метрики приложений → мониторинг/alerting + долгосрочный анализ
- **Commit log / CDC:** изменения БД → Kafka → репликация / event sourcing
- **Stream processing:** real-time обработка (Kafka Streams, Flink, Spark Streaming)
^ko-use-cases

## Карта всех тем

```
                    Kafka — обзор (Ch.1)
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     Producer       Kafka Internals   Consumer
     (Ch.3)          (Ch.6)          (Ch.4)
          │              │              │
          ▼              ▼              ▼
     Reliable      Exactly-Once    Consumer Groups
     Delivery       Semantics      & Rebalancing
     (Ch.7)          (Ch.8)          (Ch.4)
```
^ko-full-map

## Связь
- [[Kafka Internals — обзор]] — controller, replication, request processing, storage (Ch.6)
- [[Producer — архитектура]] — producer flow, конфигурация, batching (Ch.3)
- [[Partitioning]] — как ключи маппятся на партиции (Ch.3)
- [[Consumer Groups и Rebalancing]] — масштабирование потребления (Ch.4)
- [[Consumer — архитектура]] — poll loop, offset commit, seek (Ch.4)
- [[Reliable Data Delivery — обзор]] — гарантии надёжности (Ch.7)
- [[Delivery Semantics]] — at-most/at-least/exactly-once (Ch.7+8)
- [[Compaction]] — log compaction как альтернатива retention (Ch.6)
- [[Stream Processing — концепции]] — time, state, windows, joins (Ch.14)
- [[Kafka Streams — архитектура]] — topology, tasks, scaling (Ch.14)

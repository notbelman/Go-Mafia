## Быстрая навигация

- [[Kafka — обзор (Kafka The Definitive Guide)]] — что такое Kafka, зачем нужна, архитектура
- Ch.3 — Producer, Ch.4 — Consumer, Ch.6 — Internals, Ch.7 — Reliability, Ch.8 — Exactly-Once, Ch.14 — Streams

---

## Ch.1 — Intro

#### [[Kafka — обзор (Kafka The Definitive Guide)]]
Что такое Kafka, сравнение с очередями, ключевые концепции, use cases

---

## Ch.3 — Producer

#### [[Producer — архитектура]]
ProducerRecord, сериализация, partitioner, RecordAccumulator, батчинг, acks, retries

#### [[Partitioning]]
Стратегии партиционирования, sticky partitioner, custom partitioner, когда переопределять

---

## Ch.4 — Consumer

#### [[Consumer — архитектура]]
Poll loop, десериализация, offset management, auto vs manual commit, конфиги

#### [[Consumer Groups и Rebalancing]]
Consumer groups, partition assignment, eager vs cooperative rebalancing, static membership

---

## Ch.6 — Internals

#### [[Kafka Internals — обзор]]
Общая картина внутреннего устройства: контроллер, репликация, storage, request processing

#### [[Controller]]
Broker controller, leader election, partition leadership, KRaft controller quorum

#### [[KRaft]]
Переход с ZooKeeper на KRaft, metadata quorum, преимущества, архитектура

#### [[Replication]]
ISR, HW (High Watermark), leader epoch, acks=all, read-from-follower

#### [[Request Processing]]
Acceptor/Network/IO threads, request queue, zero-copy sendfile, protocol versioning

#### [[Storage — сегменты и файлы]]
Log segments, .log/.index/.timeindex файлы, active segment, retention, flush

#### [[Compaction]]
Log compaction vs retention, cleaner threads, dirty ratio, tombstone records

---

## Ch.7 — Reliable Delivery

#### [[Reliable Data Delivery — обзор]]
Committed vs uncommitted messages, гарантии брокера, committed offset vs HW

#### [[Broker Config — надёжность]]
replication.factor, min.insync.replicas, unclean.leader.election, rack awareness

#### [[Producer — надёжность]]
acks режимы, idempotent producer, retriable vs non-retriable ошибки, retries + delivery.timeout

#### [[Consumer — надёжность]]
Commit стратегии, enable.auto.commit, at-least-once vs exactly-once на стороне консьюмера

---

## Ch.8 — Exactly-Once

#### [[Delivery Semantics]]
At-most-once, at-least-once, exactly-once — сравнение, ограничения каждой семантики

#### [[Idempotent Producer]]
PID + sequence number, дедупликация на брокере, ограничения (single partition, single session)

#### [[Transactions]]
Transaction coordinator, двухфазный коммит, isolation levels, производительность

---

## Ch.14 — Stream Processing

#### [[Stream Processing — концепции]]
Потоки vs батчи, time semantics (event/processing/ingestion), state, windowing, паттерны

#### [[Kafka Streams — архитектура]]
Topology, KStream/KTable, state stores, changelog topics, задачи и потоки, vs Flink/Spark

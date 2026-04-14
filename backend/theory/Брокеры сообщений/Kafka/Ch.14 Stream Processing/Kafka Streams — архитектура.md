- **Kafka Streams** — клиентская библиотека (не фреймворк). Не нужен YARN/Mesos/K8s — просто запусти несколько instances приложения
- **Topology (DAG):** source processors → stream processors (filter, map, aggregate, join) → sink processors. Определяется в коде, выполняется на tasks
- **Task** = единица параллелизма. Количество tasks = количество партиций input topic. Каждый task обрабатывает свой subset партиций **независимо**
- Масштабирование: больше threads в одном instance ИЛИ больше instances. Tasks автоматически распределяются. Как consumer group
- **State recovery:** local state (RocksDB) + **changelog topic** (compacted). Крашнулся → replay changelog → state восстановлен. Standby replicas для быстрого failover

---

## Kafka Streams vs Flink/Spark

|  | Kafka Streams | Flink / Spark Streaming |
|---|---|---|
| **Deployment** | Библиотека (JAR) | Кластерный фреймворк |
| **Инфраструктура** | Нет (только Kafka) | YARN / K8s / standalone |
| **Масштабирование** | Запусти больше JVM | Конфигурация кластера |
| **Source/Sink** | Только Kafka | Kafka + файлы + DB + ... |
| **Exactly-once** | Kafka транзакции | Собственный механизм |
| **Сложность** | Низкая | Выше |

> [!tip] Ключевое преимущество Kafka Streams
> Нет отдельной инфраструктуры. Приложение = обычный Java/JVM процесс. Запусти 10 instances → получи кластер. ^ks-no-infra

^ks-vs-frameworks

## Topology (DAG)

```text
Source Processor → [filter] → [map] → [groupByKey] → [aggregate] → Sink Processor
       ↑                                                                    ↓
   input topic                                                        output topic
```

- **Source:** читает из Kafka topic, передаёт дальше
- **Processor:** filter, map, aggregate, join, flatMap, groupBy...
- **Sink:** пишет результат в Kafka topic

^ks-topology

**Три этапа:** ^ks-topology-steps
1. **Logical topology:** KStream/KTable + DSL операции (filter, join...)
2. **Physical topology:** `StreamsBuilder.build()` — оптимизация
3. **Execution:** `KafkaStreams.start()` — consume, process, produce

## Tasks — параллелизм

```text
Input topic: 4 partitions → 4 tasks

Instance 1 (2 threads):     Instance 2 (2 threads):
  Thread 1: Task 0 (P0)      Thread 3: Task 2 (P2)
  Thread 2: Task 1 (P1)      Thread 4: Task 3 (P3)

Каждый task:
  - подписан на свои партиции
  - выполняет ВЕСЬ topology для своих событий
  - поддерживает СВОЙ local state
  - работает НЕЗАВИСИМО от других tasks
```

^ks-tasks

> [!important]
> Макс. параллелизм = количество партиций input topic. Больше threads/instances чем партиций → лишние idle. Аналогия с consumer groups. ^ks-max-parallelism

## Repartitioning — subtopologies

```text
Topology 1:                          Topology 2:
  read(clicks) → groupBy(zipCode)      read(repartition-topic)
    → write(repartition-topic)           → aggregate(count)
                                           → write(output-topic)
```

`groupBy` меняет ключ → данные нужно перераспределить → Kafka Streams записывает в промежуточный topic → Второй subtopology читает и агрегирует → Два subtopology с разными наборами tasks → работают **независимо** (связь только через topic).

^ks-repartition

## Joins: требования к партиционированию

- **Stream-Stream Join** и **Stream-Table Join:** оба topic **должны** иметь одинаковое число партиций и **должны** быть партиционированы по join key → Kafka Streams назначает matching партиции одному task → task видит все данные для join по конкретным ключам
- **Table-Table Join:** equi-join (тот же ключ) или foreign-key join (поддерживается через KIP)

^ks-join-requirements

## State Management

```text
Application instance
  └── Task
        └── State Store (RocksDB)
              ├── in-memory + disk (spill)
              └── backed by changelog topic (compacted)

Changelog topic:
  → каждое изменение state → запись в changelog
  → compacted → хранит только последнее значение по ключу
  → при crash → replay changelog → полное восстановление state
```

^ks-state-management

## Failure Recovery

#### Без оптимизации

1. Task на Thread A крашнулся
2. Kafka Streams переназначает task на Thread B (rebalance)
3. Thread B восстанавливает state: читает changelog topic → rebuilds RocksDB state store
4. Thread B продолжает обработку

> [!warning]
> Recovery может быть **медленным** при большом state.

#### Оптимизации ^ks-failure-recovery

| Оптимизация | Описание |
|---|---|
| **Standby replicas** (`num.standby.replicas=1`) | Теневой task держит state актуальным на другом instance → failover почти мгновенный |
| **Aggressive compaction** для changelog topics | `min.compaction.lag.ms=0`, `segment.bytes=100MB` → меньше данных для replay |

## Exactly-Once в Kafka Streams

```java
// Одна строка конфигурации:
props.put("processing.guarantee", "exactly_once_v2");
```

**Под капотом:**
- Kafka Streams использует transactional producer
- Каждый task: `beginTransaction` → produce results + commit offsets → `commitTransaction`
- Atomic: результат записан + offset committed = одна транзакция
- Zombie fencing через consumer group metadata (KIP-447)

> [!note] exactly_once_v2 (Kafka 2.6+ / brokers 2.5+)
> Более эффективная реализация — один transactional producer на несколько tasks. ^ks-eo-v2

^ks-exactly-once

## Use Cases

| Use Case | Описание |
|---|---|
| **Customer service** | Обновления бронирований → все системы за секунды |
| **IoT** | Предиктивное обслуживание, паттерны в потоке датчиков |
| **Fraud detection** | Аномалии в транзакциях в реальном времени (beaconing) |
| **Real-time analytics** | Агрегации, windowed stats, enrichment |
| **Microservices** | Event-driven архитектура, local state как cache |

^ks-use-cases

## Связь
- [[Stream Processing — концепции]] — time, state, windows, joins, patterns
- [[Transactions]] — exactly-once реализован через Kafka транзакции (Ch.8)
- [[Delivery Semantics]] — exactly-once гарантии (Ch.8)
- [[Consumer Groups и Rebalancing]] — Kafka Streams использует тот же механизм (Ch.4)
- [[Compaction]] — changelog topics используют log compaction (Ch.6)
- [[Replication]] — high availability для changelog topics (Ch.6)

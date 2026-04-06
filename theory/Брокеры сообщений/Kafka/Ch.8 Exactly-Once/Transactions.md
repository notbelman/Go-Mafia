- Транзакции = **atomic multipartition write**: результат в output topic + commit offset в __consumer_offsets = **одна атомарная операция**
- `transactional.id` — уникальный, **переживает рестарты** (в отличие от PID). Broker маппит transactional.id → PID
- **Zombie fencing:** каждый `initTransactions()` инкрементирует epoch. Старый producer с тем же transactional.id но меньшим epoch → FencedProducer → не может писать
- Consumer: `isolation.level=read_committed` — не видит незакоммиченные и aborted транзакции. Default `read_uncommitted` — видит всё
- Внутри: **two-phase commit** + **transaction log** (`__transaction_state` topic) + **commit markers** в партициях

---

## Две проблемы, которые решают транзакции

#### 1. Reprocessing при крашах ^tx-problem-crash

```text
consume(record) → process → produce(result)
                                    ↓
                              commit offset
Крашнулся между produce и commit offset
→ rebalance → новый consumer перечитает record
→ result записан ДВАЖДЫ в output topic
```

#### 2. Zombie producer ^tx-problem-zombie

```text
Consumer A завис → rebalance → Consumer B получил партицию
Consumer B обрабатывает records, пишет результаты
Consumer A "проснулся" → тоже пишет результаты (он ещё не знает что zombie)
→ дубли в output topic
```

## Как транзакции решают обе проблемы

```text
Atomic multipartition write:
  BEGIN TRANSACTION
    produce(result → output topic)                              ← данные
    sendOffsetsToTransaction(offset → __consumer_offsets)       ← offset
  COMMIT TRANSACTION

→ Либо ОБА записаны, либо НИ ОДИН
→ Нет состояния "result записан, offset нет" → нет reprocessing
```

^tx-atomic-write

#### Zombie fencing через epoch ^tx-fencing

```text
Producer A (transactional.id="app-1", epoch=1) завис
Producer B стартует с transactional.id="app-1"
  → initTransactions() → epoch=2
  → pending транзакции epoch=1 aborted

Producer A "проснулся", шлёт с epoch=1
  → Broker: epoch=1 < current epoch=2
  → FencedProducerException → zombie не может писать
```

> [!tip] KIP-447 (Kafka 2.5+)
> Fencing через consumer group metadata. Не нужно один transactional.id на партицию — разные producers могут писать в одни партиции, fencing по generation consumer group. ^tx-kip447

## Consumer: isolation levels

| Уровень | Что видит |
|---|---|
| `read_uncommitted` (default) | Всё: committed, aborted, in-flight транзакции + нетранзакционные записи |
| `read_committed` | Только committed транзакции + нетранзакционные записи |

**LSO (Last Stable Offset)** — offset первой открытой транзакции. Consumer не получит данные после LSO пока транзакция не закроется → долгая открытая транзакция = задержка для consumer.

`transaction.timeout.ms = 15 min` (default) → broker абортит зависшую транзакцию.

^tx-isolation-levels

## Как работает внутри (two-phase commit)

1. **`producer.initTransactions()`** → регистрация `transactional.id` у transaction coordinator → coordinator = leader партиции `__transaction_state` для этого ID → increment epoch, abort pending транзакций

2. **`producer.beginTransaction()`** → только на клиенте, coordinator не знает

3. **`producer.send(record)`** → первый send в новую партицию → `AddPartitionsToTxnRequest` → coordinator записывает партицию в transaction log

4. **`producer.sendOffsetsToTransaction(offsets, groupMetadata)`** → coordinator → group coordinator → commit offsets

5. **`producer.commitTransaction()`** → `EndTransactionRequest` → coordinator:
   - a) Записать **INTENT TO COMMIT** в transaction log (после этого — doomed to commit, даже при крашах)
   - b) Записать commit markers во ВСЕ партиции транзакции
   - c) Записать TRANSACTION COMPLETE в transaction log

> [!note]
> Coordinator крашнулся после (a)? → Новый coordinator читает intent из лога → завершает commit.

^tx-internals

## Kafka Streams: самый простой способ

```java
// Вместо всего вышеописанного:
props.put("processing.guarantee", "exactly_once_v2");
// Всё. Kafka Streams использует транзакции под капотом.
```

^tx-kafka-streams

## Performance

| Операция | Когда |
|---|---|
| `initTransactions()` | Один раз за lifecycle producer |
| `AddPartitionsToTxn` | Один раз за партицию за транзакцию |
| Commit markers | Один на партицию |
| `initTransactions()` и commit | Синхронные (блокируют) |

- Больше messages в транзакции = меньше relative overhead
- Долгие транзакции = выше end-to-end latency (consumer ждёт)
- Consumer в `read_committed`: broker не шлёт open transactions → throughput не падает

^tx-performance

## ⚠️ Memory leak от transactional IDs

```text
Broker хранит state каждого PID/transactional.id 7 дней
(transactional.id.expiration.ms)

Создавать новый transactional.id на каждый запрос = утечка памяти!
3 новых ID/сек × 7 дней = 1.8M записей ≈ 5GB RAM на брокере
```

> [!warning]
> Reuse long-lived producers. Или снизить `transactional.id.expiration.ms`.

^tx-memory-leak

## Связь
- [[Delivery Semantics]] — транзакции = основа exactly-once
- [[Idempotent Producer]] — транзакции расширяют idempotent producer
- [[Consumer Groups и Rebalancing]] — KIP-447 fencing через consumer group metadata (Ch.4)
- [[Consumer — надёжность]] — offset commit, rebalance (Ch.7)
- [[Producer — надёжность]] — acks, retries (Ch.7)
- [[Controller]] — coordinator = leader партиции transaction log (Ch.6)

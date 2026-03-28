- Key ≠ null → **hash(key) % numPartitions** → один ключ всегда в одну партицию (пока число партиций не меняется)
- Key = null → **sticky round-robin**: заполняет батч в одну партицию, потом переключается на следующую (с Kafka 2.4)
- Добавление партиций **ломает** маппинг ключей → старые данные в одной партиции, новые в другой. Решение: создавать с запасом
- Custom partitioner: для hot keys (один ключ = 10%+ трафика) → выделить отдельную партицию
- Keys — не только для партиционирования: **compaction** (последнее значение по ключу), идентификация сообщений

---

## Зачем ключи

```
Два назначения:
1. Определяют партицию → все записи с одним ключом в одной партиции
   → один consumer обрабатывает все события одной сущности
   → гарантия порядка для одного ключа

2. Дополнительные данные (user_id, order_id, device_id)
   → используются в compaction (хранить последнее значение по ключу)
```
^part-keys-purpose

## Default Partitioner

```
key != null:
  partition = hash(key) % numPartitions
  → Kafka использует свой hash (не Java hashCode)
  → стабилен между версиями Java
  → маппинг считается по ВСЕМ партициям (не только available)
  → если партиция unavailable → ошибка (редко)

key == null:
  Sticky Round-Robin (с Kafka 2.4):
  → заполняет batch в одну партицию
  → batch полон или linger.ms истёк → переключается на следующую
  → меньше запросов, ниже latency, меньше CPU на broker
```
^part-default

## Стратегии партиционирования

```
DefaultPartitioner:     hash(key) если key != null, sticky RR если null
RoundRobinPartitioner:  всегда round-robin (даже с ключом)
UniformStickyPartitioner: всегда sticky random (даже с ключом)
Custom Partitioner:     своя логика (implements Partitioner)
```
^part-strategies

**UniformStickyPartitioner** — когда ключи нужны consumer'у (напр. ETL → primary key в БД), но нагрузка skewed. Равномерное распределение по партициям, но порядок по ключу теряется. ^part-uniform-sticky

## ⚠️ Добавление партиций ломает маппинг

```
Topic с 10 партициями:
  hash("user-123") % 10 = 3  → partition 3

Добавили до 15 партиций:
  hash("user-123") % 15 = 8  → partition 8

Старые записи user-123 в partition 3, новые в partition 8.
Consumer читает partition 3 — видит только старые.
```
^part-adding-breaks

**Решение:** создавать топик с **достаточным количеством партиций** сразу и никогда не добавлять. ^part-solution

## Custom Partitioner: hot key

```
Проблема: 10% трафика = ключ "Banana"
→ одна партиция перегружена, остальные недогружены

Решение: custom partitioner
  if key == "Banana" → last partition (выделенная)
  else → hash(key) % (numPartitions - 1)
```
^part-custom-hot-key

## Headers

```
record.headers().add("trace-id", traceId.getBytes());

Headers = метаданные записи (не часть key/value)
Use cases: tracing, routing, lineage, encryption metadata
Формат: ordered collection of String→byte[] pairs
```
^part-headers

## Связь
- [[Producer — архитектура]] — partitioner в producer flow
- [[Consumer Groups и Rebalancing]] — кол-во партиций = макс. параллелизм consumer group (Ch.4)
- [[Compaction]] — ключи используются для log compaction (Ch.6)
- [[Consumer — надёжность]] — consumer group: один consumer = subset партиций (Ch.7)
- [[Storage — сегменты и файлы]] — partition = базовая единица хранения (Ch.6)
- [[Replication]] — partition → leader + followers (Ch.6)

- Базовая единица хранения — **replica партиции**. Партиция не делится между брокерами и даже между дисками
- Партиция разбита на **сегменты**: по 1GB или 1 неделя данных (что раньше). Активный сегмент никогда не удаляется
- Формат на диске = формат по сети → **zero-copy** возможен, не нужна конвертация
- Сообщения хранятся **батчами** (с v2 формата, Kafka 0.11+). Батч = заголовок + набор записей. Overhead на запись минимален
- Два индекса: **offset → позиция в файле** и **timestamp → offset**. Повреждённый индекс — регенерируется из лога

---

## Сегменты

```
Партиция 0:
  segment-0000000000.log    (1GB, закрыт)     ← можно удалить/compactnуть
  segment-0000003500.log    (1GB, закрыт)     ← можно удалить/compactнуть
  segment-0000007200.log    (активный)         ← НИКОГДА не удаляется
```
^storage-segments

Новый сегмент создаётся при достижении лимита (1GB или `log.roll.ms`/`log.roll.hours`). Старый закрывается и становится кандидатом на удаление/compaction. ^storage-segment-roll

**Активный сегмент НЕ удаляется.** Если retention = 1 день, а сегмент содержит 5 дней данных → реально хранится 5 дней. ^storage-active-segment

Kafka держит **open file handle** на каждый сегмент каждой партиции (включая неактивные) → OS нужно тюнить на большое количество открытых файлов. ^storage-file-handles

## Формат файла (v2, с Kafka 0.11+)

```
Message Batch:
├── magic number (версия формата)
├── first offset / last offset delta
├── first timestamp / max timestamp
├── batch size (bytes)
├── leader epoch (для truncation при leader election)
├── checksum
├── attributes (16 bit): compression, timestamp type, transactional?, control?
├── producer ID, producer epoch, first sequence (для exactly-once)
└── records[]

Record:
├── size (bytes)
├── offset delta (от first offset батча)
├── timestamp delta (от first timestamp батча)
├── key, value, headers (user payload)
```
^storage-format

**Связь с exactly-once:** поля producer ID, producer epoch и first sequence в батче — основа [[Idempotent Producer]] (Ch.8). Broker валидирует sequence для дедупликации retry. ^storage-format-exactly-once

**Ключевое:** формат на диске **идентичен** формату по сети. Это позволяет zero-copy и избегает де/рекомпрессию на брокере. ^storage-format-zero-copy

**Батчинг:** producer всегда шлёт батчами (с v2). Один message = overhead батча. Два+ = экономия. Поэтому `linger.ms=10` улучшает производительность — больше шанс собрать батч. Меньше партиций → больше сообщений в батче → эффективнее. ^storage-batching

**Compression:** рекомендуется на producer. Больше батч → лучше сжатие и по сети, и на диске. ^storage-compression

## Индексы

Два индекса на каждый сегмент: ^storage-indexes

|Индекс|Маппинг|Зачем|
|:--|:--|:--|
|**offset index**|offset → позиция в файле|быстрый поиск по offset|
|**time index**|timestamp → offset|поиск по времени (Kafka Streams, failover)|

Индексы тоже разбиты на сегменты (удаляются вместе со старыми данными). При повреждении — **регенерируются** из лога (безопасно удалять вручную). ^storage-index-regen

## Tiered Storage (KIP-405)

```
                 ┌─── Local Tier ───┐     ┌─── Remote Tier ───┐
                 │ Локальные диски   │     │ HDFS / S3          │
                 │ Часы retention    │     │ Дни/месяцы retention│
                 │ Низкая latency    │     │ Высокая latency     │
                 │ Tail reads        │     │ Backfill / recovery │
                 └──────────────────┘     └────────────────────┘
```
^storage-tiered

**Зачем:** масштабировать хранилище независимо от CPU/RAM. Не нужно увеличивать кластер ради retention. Репликация/ребалансировка — только для local tier (быстрее). ^storage-tiered-why

**Изоляция:** исторические чтения (remote tier) не влияют на page cache и диск → меньше impact на real-time чтения. Без tiered storage: чтение старых данных вымывает page cache → latency p99 растёт с 21ms до 60ms. С tiered storage: 25ms vs 42ms. ^storage-tiered-isolation

## Partition Allocation

При создании топика (напр. 10 партиций, RF=3, 6 брокеров = 30 реплик): ^storage-allocation

```
Правила:
1. Равномерно распределить реплики по брокерам (30/6 = 5 на брокер)
2. Реплики одной партиции — на разных брокерах
3. Rack awareness: реплики на разных стойках (если настроено)

Алгоритм:
1. Начать с рандомного брокера (напр. 4)
2. Лидеры round-robin: P0→B4, P1→B5, P2→B0, P3→B1...
3. Followers: со смещением от лидера
   P0: leader=B4, follower1=B5, follower2=B0
```
^storage-allocation-algo

При rack awareness: брокеры упорядочиваются чередуя стойки (0,2,1,3 вместо 0,1,2,3) → каждая реплика на другой стойке. ^storage-rack-awareness

**Выбор директории на диске:** считаем партиции в каждой директории → новая партиция в директорию с наименьшим количеством. Не учитывает размер партиций и свободное место. ^storage-dir-allocation

## Связь
- [[Request Processing]] — zero-copy работает благодаря единому формату
- [[Compaction]] — как работает log compaction
- [[Replication]] — leader epoch в батче, tiered storage и репликация
- [[Kafka Internals — обзор]] — место storage в архитектуре
- [[Broker Config — надёжность]] — persistence to disk, page cache vs fsync (Ch.7)

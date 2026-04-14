- Compaction = retention policy: хранить **последнее значение для каждого ключа** (вместо удаления по времени)
- Лог делится на **clean** (уже compacted, один value на key) и **dirty** (новые записи после последнего compaction)
- **Tombstone** — сообщение с key + null value. Удаляет ключ полностью. Хранится временно, чтобы консьюмеры увидели удаление
- Compaction НЕ затрагивает **активный сегмент**. Запускается когда dirty ≥ 50% топика
- Offset map: 24 байта на запись (16-byte hash ключа + 8-byte offset) → 1GB сегмент с 1KB записями = 24MB map

---

## Три retention policy

| Policy | Поведение |
|---|---|
| `delete` | Удалять сообщения старше retention period (по умолчанию) |
| `compact` | Хранить последнее значение для каждого ключа |
| `delete.and.compact` | Комбинация: compaction + удаление по времени |

^compaction-policies

> [!warning]
> `compact` имеет смысл только для топиков с ключами. Null keys → compaction fails. ^compaction-requires-keys

## Use cases для compaction

1. **Адреса клиентов:** ключ = customer_id, value = адрес. Нужен последний адрес, не история ^compaction-usecase-address
2. **State store:** приложение пишет своё состояние в Kafka. При recovery нужно только последнее состояние ^compaction-usecase-state
3. **Changelog:** CDC, Kafka Connect — compact topic как materialized view ^compaction-usecase-changelog

## Clean и Dirty

```text
Партиция:
  ┌─────── Clean ───────┐┌─────── Dirty ─────────┐
  │ K1:v3  K2:v1  K3:v5 ││ K1:v4  K4:v1  K1:v5   │
  │ (один value на key)  ││ (новые записи)          │
  └──────────────────────┘└────────────────────────┘
                                    ▲
                              compaction выберет
                              partition с наибольшим
                              dirty/total ratio
```

^compaction-clean-dirty

## Как работает compaction

1. Compaction manager thread выбирает партицию с наибольшим dirty ratio
2. Cleaner thread читает dirty section → строит in-memory **offset map**: `{ hash(key) → latest_offset }` (24 байта на запись: 16-byte hash + 8-byte offset)
3. Читает clean segments (с самого старого):
   - key **НЕТ** в offset map → value актуален → копировать в replacement segment
   - key **ЕСТЬ** в offset map → есть более новый value → пропустить
4. Replacement segment заменяет оригинал
5. Результат: один message на каждый key (с последним value)

^compaction-flow

**Эффективность map:** 1GB сегмент, 1KB записи = 1M записей → 24MB offset map. При повторяющихся ключах — ещё меньше. ^compaction-map-efficiency

> [!note]
> Если offset map не вмещает весь dirty section — compaction берёт самые старые сегменты, остальные ждут следующего прохода. Минимум один полный сегмент должен помещаться. ^compaction-map-limit

## Tombstone — удаление ключа

```go
// Producer: удалить customer_123
producer.Send(topic, key="customer_123", value=nil)
```

^compaction-tombstone-produce

1. Cleaner видит tombstone (key + null value)
2. Normal compaction: оставляет tombstone, удаляет предыдущие values
3. Tombstone хранится настраиваемое время (чтобы консьюмеры увидели)
4. После таймаута — cleaner удаляет tombstone полностью

^compaction-tombstone-flow

> [!warning]
> Если консьюмер был offline и пропустил tombstone → он не узнает что ключ удалён. Нужно давать **достаточно времени** консьюмерам на чтение tombstone. ^compaction-tombstone-timing

## deleteRecords (AdminClient)

Отдельный механизм: сдвигает **low-water mark** → записи ниже нового LWM недоступны консьюмерам → позже удаляются cleaner thread. Работает и для обычных топиков, и для compacted. ^compaction-delete-records

## Timing

| Параметр | Описание |
|---|---|
| Активный сегмент | НИКОГДА не compactится (как и с delete policy) |
| `min.compaction.lag.ms` | Минимум времени после записи до compaction |
| `max.compaction.lag.ms` | Максимум задержки до compaction (напр. GDPR: 30 дней) |
| Dirty ratio ≥ 50% | Запуск compaction (настраивается) |

^compaction-timing

## Связь
- [[Storage — сегменты и файлы]] — сегменты, формат файлов, индексы
- [[Kafka Internals — обзор]] — место compaction в архитектуре
- [[Replication]] — compaction и репликация
- [[Partitioning]] — ключи используются и для партиционирования, и для compaction (Ch.3)
- [[Kafka Streams — архитектура]] — changelog topics используют compaction для state recovery (Ch.14)

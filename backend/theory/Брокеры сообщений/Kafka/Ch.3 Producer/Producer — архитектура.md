- Producer flow: ProducerRecord → serialize → partitioner → **batch** (по topic+partition) → sender thread → broker → response/error
- Три способа отправки: **fire-and-forget** (потеря возможна), **sync** (send().get(), медленно), **async** (send + callback, рекомендуется)
- Ключевые конфиги производительности: `linger.ms` (задержка для сбора батча), `batch.size` (размер батча в байтах), `compression.type`, `buffer.memory`
- **delivery.timeout.ms** (default 2 min) — общий таймаут на доставку. Включает retries. Рекомендуется вместо ручной настройки retries
- Сериализация: рекомендуется Avro/Protobuf + **Schema Registry** (схема хранится отдельно, в записи только schema ID)

---

## Producer Flow

```text
Application
    │
    ▼
ProducerRecord(topic, [key], value, [headers])
    │
    ▼
Serializer (key + value → byte[])
    │
    ▼
Partitioner (выбирает партицию, если не указана явно)
    │
    ▼
Record Buffer (batch по topic+partition)
    │
    ▼
Sender Thread (отдельный поток, шлёт батчи на брокеры)
    │
    ▼
Broker → response (RecordMetadata) или error
    │
    ▼
Callback (если async)
```

^pa-flow

## Три способа отправки

```java
// Fire-and-forget: потеря возможна
producer.send(record);

// Sync: блокирует, медленно, НЕ для production
producer.send(record).get();

// Async + callback: рекомендуется
producer.send(record, (metadata, exception) -> {
    if (exception != null) handleError(exception);
});
```

^pa-send-methods

> [!note]
> **Callback** выполняется в main thread producer'а → должен быть быстрым, без блокирующих операций. ^pa-callback-thread

## Конфигурация: delivery time

```text
         ┌─── max.block.ms ───┐┌──────── delivery.timeout.ms ────────┐
         │ send() блокирован   ││ от помещения в batch до ответа/fail │
         │ (буфер полон или    ││                                      │
         │  нет метаданных)    ││  linger.ms  request.timeout  retries │
         └─────────────────────┘└──────────────────────────────────────┘
```

^pa-delivery-time

|Параметр|Default|Что делает|
|:--|:--|:--|
|`max.block.ms`|60s|Макс. время блокировки send() (буфер полон / нет metadata)|
|`delivery.timeout.ms`|120s|Общий таймаут: batch ready → broker response. Включает все retries|
|`request.timeout.ms`|30s|Таймаут одного запроса к broker (без retries)|
|`linger.ms`|0|Задержка перед отправкой батча. >0 = больше в батче = выше throughput|
|`retries`|MAX_INT|Сколько раз retry. Лучше не трогать, управлять через delivery.timeout.ms|
|`retry.backoff.ms`|100ms|Пауза между retries|

^pa-timeout-configs

> [!tip] Рекомендация
> Настраивать `delivery.timeout.ms` вместо retries. "Leader election занимает ~30s → ставим delivery.timeout.ms=120000". ^pa-timeout-recommendation

## Конфигурация: производительность

|Параметр|Default|Что делает|
|:--|:--|:--|
|`batch.size`|16KB|Размер батча **в байтах** (не в сообщениях). Больше = эффективнее, но больше памяти|
|`linger.ms`|0|0 = шлём сразу. 10+ = ждём, собираем батч. **Сильно улучшает throughput**|
|`compression.type`|none|snappy (быстро), gzip (лучше сжатие), lz4, zstd. Рекомендуется включать|
|`buffer.memory`|32MB|Общий буфер для всех батчей. Переполнен → send() блокируется на max.block.ms|
|`max.in.flight.requests`|5|Сколько батчей в полёте без ответа. Больше = выше throughput. С idempotence ≤5|

^pa-perf-configs

**linger.ms + batch.size:** linger.ms > 0 увеличивает шанс собрать полный батч → лучше compression, меньше запросов, выше throughput. Trade-off: +latency на linger.ms мс. ^pa-linger-batch

**compression:** рекомендуется на producer. Формат на диске = формат по сети → broker не пережимает. Больше батч → лучше сжатие. ^pa-compression

## Ordering Guarantees

```text
retries > 0 + max.in.flight > 1 → возможен reorder:
  Batch 1 failed, Batch 2 in-flight succeeded, Batch 1 retry succeeded
  → порядок: Batch 2, Batch 1 (перепутан!)
```

> [!tip] Решение
> `enable.idempotence=true` — гарантирует порядок при max.in.flight ≤ 5 + дедупликация retries

^pa-ordering

## Сериализация

| Вариант | Описание |
|---|---|
| JSON | Простой, но verbose, нет schema enforcement |
| **Avro / Protobuf** | Рекомендуется. Компактный, schema evolution |
| Custom | Антипаттерн — хрупкий, не поддерживает schema evolution |

#### Schema Registry

```text
Producer: serialize(data, schema) → [schema_id + binary_data]
Consumer: [schema_id + binary_data] → fetch schema by ID → deserialize
```

Зачем: schema evolution (добавить/удалить поле без поломки), одна схема на все producer/consumer, версионирование.

^pa-serialization

> [!warning] Custom serializer = антипаттерн
> Хрупкий, не поддерживает schema evolution, все producer/consumer должны синхронно обновляться. ^pa-custom-serializer-bad

## Quotas и Throttling

Broker может ограничить rate producer'а (bytes/sec). При превышении — broker задерживает ответы → producer автоматически замедляется. Метрики: `produce-throttle-time-avg/max`. ^pa-quotas

## Связь
- [[Producer — надёжность]] — acks, retries, error handling (Ch.7)
- [[Idempotent Producer]] — enable.idempotence, PID, sequence (Ch.8)
- [[Partitioning]] — как выбирается партиция
- [[Request Processing]] — что делает broker с produce request (Ch.6)
- [[Storage — сегменты и файлы]] — batch format на диске = по сети (Ch.6)

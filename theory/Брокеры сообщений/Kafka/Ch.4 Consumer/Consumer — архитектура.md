- Poll loop — сердце consumer'а. `poll()` делает всё: получение данных, heartbeat, rebalance, вызов callbacks
- **Thread safety:** один consumer = один thread. Для параллелизма: несколько consumer'ов в разных потоках ИЛИ один consumer + worker thread pool
- **Offset commit:** в `__consumer_offsets` topic. commitSync (блокирует, retry), commitAsync (не блокирует, без retry), комбинация обоих
- **RebalanceListener:** `onPartitionsRevoked` — commit offsets перед потерей партиции. `onPartitionsAssigned` — подготовка к новым
- **Seek:** можно начать с любого offset (`seekToBeginning`, `seekToEnd`, `seek(partition, offset)`, поиск по timestamp)

---

## Poll Loop

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        process(record);
    }
    consumer.commitSync();   // или commitAsync()
}
```

^ca-poll-loop

**poll() делает больше чем кажется:** ^ca-poll-internals
- Первый вызов: находит GroupCoordinator, join group, получает partition assignment
- Каждый вызов: шлёт heartbeat, обрабатывает rebalance, вызывает RebalanceListener
- Если `poll()` не вызван `max.poll.interval.ms` → consumer выгоняют из группы

> [!warning]
> Не блокировать poll loop! Длинная обработка → увеличить `max.poll.interval.ms` или уменьшить `max.poll.records`. ^ca-poll-no-block

## Ключевые конфигурации

|Параметр|Default|Что делает|
|:--|:--|:--|
|`fetch.min.bytes`|1 byte|Мин. данных для ответа. Больше → меньше запросов, выше latency|
|`fetch.max.wait.ms`|500ms|Макс. ожидание при fetch.min.bytes. Trade-off с latency|
|`fetch.max.bytes`|50MB|Макс. размер ответа. Лимит памяти consumer'а|
|`max.poll.records`|500|Макс. записей за один poll(). Контроль объёма обработки|
|`max.partition.fetch.bytes`|1MB|Макс. данных на партицию. Лучше использовать fetch.max.bytes|
|`session.timeout.ms`|10s|Таймаут heartbeat → rebalance|
|`max.poll.interval.ms`|5 min|Таймаут между poll() → consumer evicted|
|`offsets.retention.minutes`|7 дней|Broker хранит committed offsets. Группа пустует дольше → offsets удалены → как новая группа|

^ca-configs

## Offset Commit — три способа

| Способ | Поведение | Когда использовать |
|---|---|---|
| `commitSync()` | Блокирует до ответа broker'а. Retry при retriable errors | Безопасно, но медленно |
| `commitAsync()` | Не блокирует, не retry'ит | Быстро, но без гарантии |
| **Комбинация** | commitAsync() в цикле + commitSync() при shutdown | Рекомендуется |

^ca-commit-methods

> [!warning] commitAsync + retry
> Retry опасен: commit offset 2000 после commit offset 3000 → откат. Можно с callback для логирования.

**Commit specific offset:** можно коммитить offset конкретной партиции в середине батча. Полезно для больших батчей — не перечитывать всё при крэше. ^ca-commit-specific

```java
// Коммит каждые 1000 записей:
currentOffsets.put(new TopicPartition(record.topic(), record.partition()),
    new OffsetAndMetadata(record.offset() + 1, null));  // +1!
if (count % 1000 == 0)
    consumer.commitAsync(currentOffsets, null);
```

^ca-commit-specific-example

## RebalanceListener

```java
consumer.subscribe(topics, new ConsumerRebalanceListener() {

    void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        // Eager: вызывается ДО rebalance, ВСЕ партиции
        // Cooperative: вызывается ПОСЛЕ, только отзываемые партиции
        consumer.commitSync(currentOffsets);  // сохранить прогресс!
    }

    void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        // Подготовка: загрузить state, seek если нужно
        // Cooperative: может быть вызван с пустой коллекцией
    }

    void onPartitionsLost(Collection<TopicPartition> partitions) {
        // Только cooperative, exceptional case
        // Партиции уже у нового owner'а — аккуратно с state!
    }
});
```

^ca-rebalance-listener

## Seek — чтение с произвольного offset

```java
// С начала:
consumer.seekToBeginning(consumer.assignment());

// С конца:
consumer.seekToEnd(consumer.assignment());

// По timestamp (например, час назад):
Map<TopicPartition, Long> timestamps = consumer.assignment().stream()
    .collect(Collectors.toMap(tp -> tp, tp -> oneHourAgo));
Map<TopicPartition, OffsetAndTimestamp> offsets = consumer.offsetsForTimes(timestamps);
offsets.forEach((tp, ots) -> consumer.seek(tp, ots.offset()));
```

^ca-seek

**Use cases:** recovery после потери данных, skip ahead при отставании, replay с определённого момента. ^ca-seek-usecases

## Standalone Consumer (без группы)

```java
// Без subscribe() — напрямую assign партиции:
List<TopicPartition> partitions = consumer.partitionsFor("topic").stream()
    .map(pi -> new TopicPartition(pi.topic(), pi.partition()))
    .collect(Collectors.toList());
consumer.assign(partitions);
// Нет rebalance, нет consumer group coordination
// Но новые партиции не подхватятся автоматически
```

^ca-standalone

**assign vs subscribe:** нельзя использовать оба. assign = ручное управление, subscribe = автоматическое через группу. ^ca-assign-vs-subscribe

## Thread Safety

Один consumer = один thread (строгое правило). ^ca-thread-safety

**Варианты параллелизма:**

1. **Несколько consumer'ов в разных потоках** (ExecutorService) — каждый в своей группе или в одной группе
2. **Один consumer → queue → worker threads** — consumer читает, workers обрабатывают. Сложнее с offset commit (нужно ждать workers)

## consumer.wakeup() — graceful shutdown

```java
// Единственный thread-safe метод consumer'а
// Из shutdown hook:
Runtime.getRuntime().addShutdownHook(new Thread(() -> consumer.wakeup()));

// В poll loop:
try {
    while (true) { consumer.poll(...); ... }
} catch (WakeupException e) {
    // ignore — это сигнал к завершению
} finally {
    consumer.close();  // commit offsets + leave group → немедленный rebalance
}
```

^ca-wakeup

## Связь
- [[Consumer Groups и Rebalancing]] — группы, rebalance, assignment strategies (Ch.4)
- [[Consumer — надёжность]] — reliability patterns, retry, monitoring (Ch.7)
- [[Delivery Semantics]] — at-most/at-least/exactly-once (Ch.8)
- [[Transactions]] — isolation.level, read_committed (Ch.8)
- [[Request Processing]] — fetch request, zero-copy, fetch session cache (Ch.6)

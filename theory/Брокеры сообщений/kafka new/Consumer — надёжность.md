- Консьюмер читает только **committed** данные — consistency гарантирована Kafka. Задача консьюмера — **не потерять offset**
- Главная причина потери данных на consumer: commit offset для сообщений, которые **прочитаны но НЕ обработаны**
- `enable.auto.commit=true` — удобно, но нет контроля над дублями. Если обработка в poll loop — безопасно. Если async — опасно
- `auto.offset.reset=earliest` — безопаснее (дубли лучше потери). `latest` — пропустит сообщения
- При rebalance — коммитить offsets **до** отзыва партиций. При retry — **не коммитить** offset неудачных записей

---

## Почему consumer теряет данные

```
1. Consumer читает batch [offset 100-110]
2. Обработал 100-105, не успел 106-110
3. Auto-commit коммитит offset 110
4. Consumer крашится
5. Новый consumer начинает с offset 110
6. Offsets 106-110 ПОТЕРЯНЫ (committed но не обработаны)
```
^cons-data-loss-scenario

**Правило #1:** коммитить offset **только после** полной обработки сообщений. ^cons-rule-1

## Четыре ключевых конфигурации

|Параметр|Что делает|Рекомендация|
|:--|:--|:--|
|`group.id`|Группа консьюмеров. Одинаковый → партиции делятся между ними|Уникальный если нужно видеть ВСЕ сообщения|
|`auto.offset.reset`|Что делать без сохранённого offset|`earliest` — безопаснее (дубли > потери)|
|`enable.auto.commit`|Автоматический commit по расписанию|`false` если обработка за пределами poll loop|
|`auto.commit.interval.ms`|Частота auto-commit (default 5s)|Чаще = меньше дублей, но больше overhead|
^cons-four-configs

## Auto-commit: когда безопасно, когда нет

```
БЕЗОПАСНО (auto-commit=true):
  while true:
    records = poll()
    for record in records:
      process(record)    ← вся обработка ВНУТРИ poll loop
    // auto-commit сработает при следующем poll()
    // все records гарантированно обработаны

ОПАСНО (auto-commit=true):
  while true:
    records = poll()
    for record in records:
      executor.submit(record)  ← обработка в ДРУГОМ потоке
    // auto-commit коммитит offset
    // но executor может ещё не закончить → потеря
```
^cons-auto-commit-safety

## Manual commit — важные принципы

**1. Коммитить ПОСЛЕ обработки, не после чтения** ^cons-manual-after-processing
```
// ПРАВИЛЬНО:
records = poll()
process(records)
consumer.commitSync()

// НЕПРАВИЛЬНО:
records = poll()
consumer.commitSync()  ← коммит ДО обработки
process(records)       ← крашнемся тут → данные потеряны
```

**2. Коммитить правильный offset** ^cons-manual-right-offset
```
// ОШИБКА: коммит последнего ПРОЧИТАННОГО offset
// вместо последнего ОБРАБОТАННОГО
// Если обрабатываем порциями внутри loop → коммитить
// offset конкретной обработанной порции
```

**3. Частота commit = trade-off** ^cons-manual-frequency
```
Каждое сообщение → максимальная надёжность, overhead высокий
Каждый poll      → баланс
Каждые N polls   → меньше overhead, больше дублей при крашах

Commit = produce с acks=all в __consumer_offsets topic
→ все offset commits одной consumer group → один брокер
→ может стать bottleneck
```

## Rebalance

При rebalance партиции перераспределяются между консьюмерами. Нужно: ^cons-rebalance
- **Перед потерей партиции:** коммитить текущие offsets
- **При получении новой партиции:** очистить локальный state

## Consumer retry patterns

```
Проблема: record #30 не обработан, #31 обработан.
          Нельзя коммитить offset 31 — иначе 30 считается обработанным.

Паттерн 1 — pause + buffer:
  1. Коммитить последний успешный offset (29)
  2. Сохранить необработанные records в буфер
  3. consumer.pause() — новые poll() не вернут данных
  4. Retry обработку из буфера
  5. Успех → consumer.resume()

Паттерн 2 — retry topic (dead-letter queue):
  1. Записать failed record в отдельный retry-topic
  2. Коммитить offset основного топика (продолжить)
  3. Отдельная consumer group обрабатывает retry-topic
  4. Или один consumer подписан на оба, retry-topic паузится между попытками
```
^cons-retry-patterns

## Consumer с состоянием

```
Проблема: считаем moving average. При рестарте нужен и offset, и state.

Решение: писать accumulated value в results topic
         одновременно с commit offset.
         При старте: прочитать последний result → продолжить.

Лучшее решение: Kafka Streams / Flink — встроенное state management.
```
^cons-stateful

## Мониторинг consumer

```
Главная метрика: consumer lag
  = последний committed offset на брокере - последний consumed offset

Идеал: lag ≈ 0 (колеблется из-за batch processing)
Проблема: lag растёт → consumer не справляется

Burrow (LinkedIn) — специализированный мониторинг consumer lag.
Kafka timestamp (с 0.10.0) → можно считать end-to-end latency
  (produce time → consume time)
```
^cons-monitoring

## Связь
- [[Reliable Data Delivery — обзор]] — общие гарантии
- [[Consumer Groups и Rebalancing]] — группы, rebalance, assignment strategies (Ch.4)
- [[Consumer — архитектура]] — poll loop, конфиги, commit APIs, seek (Ch.4)
- [[Producer — надёжность]] — producer-side надёжность
- [[Broker Config — надёжность]] — min.insync.replicas влияет на committed
- [[Delivery Semantics]] — at-most/at-least/exactly-once (Ch.8)
- [[Transactions]] — isolation.level, read_committed, LSO (Ch.8)
- [[Replication]] — high-water mark определяет что видит consumer (Ch.6)
- [[Request Processing]] — fetch request flow, zero-copy (Ch.6)

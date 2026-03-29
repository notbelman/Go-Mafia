- **Stream processing** = непрерывная обработка unbounded dataset. Между request-response (мс) и batch (часы). Реагируем за секунды/минуты
- **Event stream:** упорядоченный, immutable, replayable. Kafka хранит потоки долго → replay возможен (в отличие от TCP)
- **Stream-Table Duality:** stream = лог изменений, table = текущее состояние. Stream → table (materialize: replay событий). Table → stream (CDC)
- **Time:** event time (когда произошло) vs log append time (когда попало в Kafka) vs processing time (когда обработано). Event time — главный
- **Windows:** tumbling (неперекрывающиеся), hopping (перекрывающиеся), session (по активности). Grace period — сколько ждать опоздавшие события

---

## Три парадигмы обработки данных

```
Request-Response:  < 1ms–100ms    Блокирующий      OLTP, API
Batch Processing:  часы–дни       По расписанию     ETL, DWH, Hadoop
Stream Processing: мс–минуты     Непрерывный       Real-time analytics, alerting
```
^sp-paradigms

Stream processing заполняет промежуток: не нужен ответ за миллисекунды, но нельзя ждать до завтра. ^sp-gap

## Event Stream — три свойства

```
1. Ordered       — события имеют порядок (offset, timestamp)
2. Immutable     — события нельзя изменить, только добавить новые
                   (отмена = новое событие, не удаление старого)
3. Replayable    — можно перечитать с любого offset
                   (ключевое преимущество Kafka для stream processing)
```
^sp-stream-properties

## Stream-Table Duality

```
Stream (лог изменений):          Table (текущее состояние):
  "Пришли красные туфли +300"      Красные: 299
  "Синие проданы -1"               Синие: 0
  "Красные проданы -1"             Зелёные: 0
  "Синие возвращены +1"
  "Зелёные проданы -1"

Stream → Table: materialize (replay всех событий → получить состояние)
Table → Stream: CDC (Change Data Capture — ловить изменения в таблице)
```
^sp-stream-table-duality

**На собесе:** "Kafka topic — это stream. Compacted topic — это table. KTable в Kafka Streams — materialized view из stream." ^sp-duality-interview

## Time

```
Event time:       когда событие ПРОИЗОШЛО (поле в записи или producer timestamp)
                  → самый важный для stream processing
                  → "сколько продаж было в 14:00?" — event time

Log append time:  когда Kafka ПОЛУЧИЛ запись (ingestion time)
                  → менее релевантен, но стабилен

Processing time:  когда приложение ОБРАБОТАЛО запись
                  → ненадёжен: зависит от задержек, нагрузки, порядка чтения
                  → лучше избегать
```
^sp-time

**Kafka Streams:** TimestampExtractor — выбирает какой time использовать. Можно извлекать timestamp из содержимого записи. ^sp-timestamp-extractor

## Windows

```
┌─── Tumbling (неперекрывающееся) ───┐
│ [00:00─00:05] [00:05─00:10] [00:10─00:15] │
│  advance = size, без пропусков и overlap    │
└────────────────────────────────────────────┘

┌─── Hopping (перекрывающееся) ───┐
│ size=5min, advance=1min:         │
│ [00:00─00:05]                    │
│   [00:01─00:06]                  │
│     [00:02─00:07]                │
│ Событие попадает в несколько окон│
└──────────────────────────────────┘

┌─── Session (по активности) ───┐
│ gap=30s: нет событий 30s → новая сессия │
│ Размер сессии определяется данными       │
└──────────────────────────────────────────┘

Grace period: сколько ждать поздние события
  → событие с event time внутри окна, но пришло позже
  → grace period не истёк → обновить результат окна
  → grace period истёк → событие отброшено
```
^sp-windows

## Joins

```
Stream-Table Join:
  Каждое событие из stream обогащается данными из table
  → lookup по ключу в materialized table
  → аналог JOIN fact + dimension в DWH
  → нет окна — table всегда актуальная

Stream-Stream Join (windowed):
  Два потока событий, join по ключу В ПРЕДЕЛАХ окна
  → "клик произошёл в течение 1 сек после поиска"
  → оба потока хранятся в state store в пределах окна
  → требует одинаковое число партиций и партиционирование по join key

Table-Table Join:
  Два materialized state, join текущего состояния
  → nonwindowed, equi-join по ключу
  → Kafka Streams поддерживает foreign-key join
```
^sp-joins

## State

```
Local (internal) state:
  → embedded RocksDB в Kafka Streams
  → быстро, но ограничен памятью/диском
  → изменения пишутся в changelog topic (compacted) → recovery
  → при rebalance: state восстанавливается из changelog

External state:
  → Cassandra, Redis, PostgreSQL
  → безлимитный размер, но +latency, +complexity
  → stream processing старается избегать external state
```
^sp-state

## Out-of-Sequence Events

```
Мобилка потеряла WiFi на 2 часа → переподключилась → шлёт 2 часа событий
IoT датчик был offline → отправил пакет за неделю

Что делать:
1. Определить: событие out of sequence? (event time < текущее время)
2. Задать grace period: обновлять окна если задержка < порог
3. Обновить результат: Kafka Streams пишет в compacted topic
   → новое значение для ключа окна перезаписывает старое
4. Слишком старые события → отбросить
```
^sp-out-of-sequence

## Design Patterns

```
Single-Event (map/filter):   без state, каждое событие независимо
                             → фильтрация, трансформация формата
                             → легко масштабировать, легко recover

Local State (aggregation):   group-by + aggregate с local state
                             → min/max/avg/count по окнам
                             → state в RocksDB + changelog topic

Repartitioning:              нужен другой ключ для aggregation
                             → записать в промежуточный topic с новым ключом
                             → вторая подтопология читает и агрегирует
                             → topology разбивается на subtopologies

Stream-Table Join:           обогащение потока данными из таблицы
                             → CDC → KTable → join с KStream

Stream-Stream Join:          два потока, windowed join
                             → одинаковое число партиций, один join key
```
^sp-design-patterns

## Связь
- [[Kafka Streams — архитектура]] — topology, tasks, scaling, exactly-once
- [[Delivery Semantics]] — exactly-once в stream processing через транзакции (Ch.8)
- [[Transactions]] — Kafka Streams использует транзакции под капотом (Ch.8)
- [[Compaction]] — changelog topics для state recovery (Ch.6)
- [[Consumer Groups и Rebalancing]] — Kafka Streams использует consumer groups (Ch.4)
- [[Kafka — обзор (Kafka The Definitive Guide)]] — stream processing как use case Kafka (Ch.1)

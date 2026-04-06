- Kafka использует **бинарный протокол** (TCP). 61 тип запросов. Клиенты всегда инициируют соединение
- Архитектура: acceptor thread → processor threads (network) → request queue → I/O threads (handler) → response queue
- **Produce:** запрос идёт к лидеру партиции. acks=0/1 → ответ сразу. acks=all → **purgatory** (ждём ISR)
- **Fetch:** лидер отдаёт данные через **zero-copy** (из файла/page cache прямо в сокет, без промежуточных буферов)
- Консьюмеры читают только до **high-water mark** (committed). Запрос не того брокера → "Not a Leader for Partition"

---

## Архитектура обработки запросов

```text
Client → Acceptor Thread → Processor Thread (network) → Request Queue
                                                              │
                                                              ▼
Client ← Processor Thread ← Response Queue ← I/O Thread (handler)
                                                    │
                                                    ▼
                                              Purgatory
                                         (отложенные ответы:
                                          acks=all, DeleteTopic)
```

^rp-architecture

| Компонент | Роль |
|---|---|
| **Acceptor thread** | По одному на каждый порт. Принимает соединение, передаёт processor thread |
| **Processor threads** | Забирают запросы из соединений → кладут в request queue. Забирают ответы → отправляют клиентам |
| **I/O threads** | Забирают из request queue, обрабатывают, кладут результат в response queue |
| **Purgatory** | Буфер для отложенных ответов (acks=all, admin операции) |

^rp-architecture-table

## Заголовок запроса

Каждый запрос содержит: ^rp-header
- **Request type (API key)** — тип запроса
- **Request version** — для совместимости между версиями
- **Correlation ID** — уникальный ID для матчинга запрос↔ответ и логирования
- **Client ID** — идентификатор приложения

## Маршрутизация запросов

Produce и Fetch **обязаны** идти к лидеру партиции. Не тот брокер → ошибка `Not a Leader for Partition`. ^rp-routing

1. Клиент шлёт Metadata запрос (любому брокеру) → получает: партиции, реплики, кто лидер
2. Клиент кэширует метаданные
3. Produce/Fetch → напрямую к нужному брокеру
4. Периодически обновляет (`metadata.max.age.ms`)
5. При ошибке `Not a Leader` → обновляет метаданные немедленно

^rp-metadata-flow

## Produce Request

1. Запрос приходит к лидеру партиции
2. Валидация: права на запись? acks валидный? Если acks=all: достаточно ли ISR?
3. Запись на диск (в filesystem cache, **НЕ** на диск напрямую — Kafka не ждёт fsync, надёжность через репликацию)
4. `acks=0` или `acks=1` → ответ сразу. `acks=all` → запрос в purgatory → ответ после репликации на все ISR

^rp-produce-flow

> [!note]
> Сообщения пишутся в **filesystem cache**, не на диск. Kafka полагается на репликацию, а не на fsync, для durability. ^rp-produce-no-fsync

## Fetch Request

1. Клиент: "дай сообщения начиная с offset 53 в partition 0"
2. Валидация: offset существует? (слишком старый — удалён, слишком новый — нет ещё)
3. Чтение из партиции (до лимита клиента)
4. Отправка через **zero-copy**

^rp-fetch-flow

> [!tip] Zero-copy
> Данные из файла (page cache) → прямо в сокет, **без промежуточных буферов** в памяти. В обычных БД: файл → буфер приложения → сокет. Zero-copy убирает копирование → значительный прирост производительности. ^rp-zero-copy

#### Лимиты fetch ^rp-fetch-limits

- **Верхний** — максимум данных на партицию (клиент должен выделить память под ответ)
- **Нижний** — минимум данных (напр. 10KB). Брокер ждёт пока накопится → меньше запросов, меньше overhead
- **Timeout** — если минимум не набрался за X мс → отдать что есть

**Fetch session cache:** для консьюмеров с большим количеством партиций. Кэшируется список партиций → incremental fetch (не посылать весь список каждый раз). ^rp-fetch-session

## Версионирование протокола

Новые брокеры понимают старые запросы, но **не наоборот**.

> [!warning]
> Сначала обновляй **брокеры**, потом **клиентов**. ^rp-versioning

`ApiVersionRequest` — клиент спрашивает брокер, какие версии запросов поддерживаются. ^rp-api-version

## Связь
- [[Replication]] — ISR, high-water mark, acks=all → purgatory
- [[Storage — сегменты и файлы]] — формат файлов, zero-copy работает потому что формат на диске = формат по сети
- [[Kafka Internals — обзор]] — место request processing в архитектуре
- [[Producer — надёжность]] — acks, retries, как produce request обрабатывается (Ch.7)
- [[Consumer — надёжность]] — fetch, offset commit, consumer lag (Ch.7)

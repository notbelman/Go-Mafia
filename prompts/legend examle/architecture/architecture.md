# TMS ЛАНИТ — System Design

**Система управления логистикой** для федерального оператора грузоперевозок. Real-time трекинг 15 000 ТС, 5 внешних провайдеров телематики, стейт-машина заказов.

**ASR:** write-intensive телематический pipeline (~15 000 GPS events/sec) + strong consistency заказов.
**Стек:** Go, gRPC, Kafka, PostgreSQL, Redis, ClickHouse, K8s.

---

## Моя зона ответственности

Подробная документация:
- [[prompts/legend examle/architecture/my_part_adapters]] — 5 адаптеров + Telematics Service (валидация, дедупликация, роутинг)
- [[prompts/legend examle/architecture/adapters_deep_dive]] — Deep dive по каждому адаптеру (edge cases, failure modes, протоколы)
- [[prompts/legend examle/architecture/my_part_tracking]] — Tracking Service (Redis + PG + S3, hot/warm/cold)

---

## Архитектура

```
Клиенты → LB (NGINX L7) → API Gateway (gRPC-Gateway) → Сервисы (gRPC) ↔ Kafka
```

### Телематический pipeline (мой scope)

```
┌─────────── ПРОВАЙДЕРЫ ───────────┐
│ Navixy(WS) СКАУТ(REST) Omnicomm │
│ (Webhook) Wialon(TCP) АвтоГРАФ  │
│ (SOAP)                          │
└──────────────┬───────────────────┘
               ▼
     5 адаптеров (stateless)
     формат провайдера → protobuf
     1 пакет трекера → 1-3 события
               │
               ▼
     Kafka: telematics.raw (12 партиций, key=vehicle_id)
               │
               ▼
     Telematics Service (×3-5, in-memory дедупликация)
     валидация → дедупликация → роутинг
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
    .gps    .engine  .driver-events
   15K/s    500/s     30/s
       │       │       │
       ▼       ▼       ▼
  Tracking Analytics Notification
```

### Все сервисы

| Сервис | TLDR | Чей |
|---|---|---|
| **5 адаптеров** | WS, REST, Webhook, TCP binary, SOAP → protobuf → Kafka | Мой |
| **Telematics** | Валидация, in-memory дедупликация (sharded map), роутинг в 3 топика | Мой |
| **Tracking** | Redis (hot, текущие позиции) + PG (warm, 30 дней) + S3 (cold, архив). Batch INSERT, WebSocket push | Мой |
| **Order** | Стейт-машина 9 состояний, transaction outbox → Kafka, sync репликация PG | Алексей (тимлид) |
| **Routing** | Расчёт ETA, distance matrix, selective Kafka processing | Артём |
| **Analytics** | ClickHouse, CQRS, предагрегация, дашборды | Максим |
| **Notification** | Event-driven (Kafka consumer). Push/email/webhook | Максим |
| **Auth** | JWT RS256, роли (client/dispatcher/driver/admin) | Общий |
| **Report** | Отложенная генерация PDF/Excel через Kafka, presigned URL из S3 | Общий |

### Стейт-машина заказа

```
CREATED → CONFIRMED → DRIVER_ASSIGNED → LOADING → IN_TRANSIT → UNLOADING → DELIVERED → COMPLETED
```

Любое не-финальное → CANCELLED.

---

## Инфраструктура

| Компонент | TLDR |
|---|---|
| **API Gateway** | gRPC-Gateway, REST endpoints, JWT валидация, rate limiting |
| **Load Balancer** | NGINX Ingress, L7, Round Robin, SSL termination, WebSocket |
| **Kafka** | 3 брокера (Strimzi), replication factor 3, at-least-once + идемпотентность |
| **PostgreSQL** | Партиционирование по месяцам, async реплика (tracking), sync реплика (orders), Patroni |
| **Redis** | Sentinel, текущие позиции (15K SET/sec), дедупликация заменена на in-memory |
| **ClickHouse** | OLAP — Analytics Service. Агрегации, дашборды, отчёты |
| **S3 (MinIO)** | Cold storage GPS-истории, отчёты, бэкапы PG |

---

## Ключевые паттерны

| Паттерн | Где | Зачем |
|---|---|---|
| Adapter Pattern | 5 адаптеров | Единый protobuf из 5 гетерогенных протоколов |
| In-memory Sharded Map | Telematics Service | Дедупликация 15K events/sec без Redis |
| Batch INSERT | Tracking Service | 15K write/sec → 5 batch/sec, ON CONFLICT DO NOTHING |
| Hot/Warm/Cold | Tracking Service | Redis (3 МБ) → PG (6.6 TB, 30 дней) → S3 (архив) |
| Transaction Outbox | Order Service | At-least-once доставка событий заказов в Kafka |
| Circuit Breaker | Адаптеры | Защита от ненадёжных провайдеров |
| CQRS | Analytics Service | Разные модели для write (Kafka) и read (ClickHouse) |
| Idempotency | Tracking, Telematics | At-least-once Kafka + идемпотентные consumer-ы |

---

## Таблица решений (топ)

| Решение | Почему |
|---|---|
| Kafka как шина событий | 15K events/sec, fan-out на 3+ consumer-а, replay, at-least-once |
| In-memory дедупликация (не Redis) | Kafka партиционирует по vehicle_id → дубли всегда в том же инстансе |
| PG, не ClickHouse (для Tracking) | Point queries, команда знает PG. ClickHouse → Analytics Service |
| Batch INSERT (не COPY) | ON CONFLICT нативно, проще. COPY как план B |
| Async репликация PG (Tracking) | GPS — не деньги. Потеря 1-2 сек при failover ок |
| Sync репликация PG (Orders) | Потеря заказа недопустима |
| Hot/Warm/Cold storage | 237 TB за 3 года в одном PG нереально. 30 дней warm + S3 cold |
| Protobuf (не JSON) | 15K events/sec, компактнее в 2-3x, типизация, Schema Registry |
| Redis TTL 5 мин | Dead man switch — нет данных 5 мин = ТС "офлайн" на карте |
| context.WithTimeout на всё | Инцидент: зависание горутин при GC pause Redis |

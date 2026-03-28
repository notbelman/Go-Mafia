- Даже при идеальной конфигурации брокеров, **producer может потерять данные** если неправильно настроен
- **acks=all** — единственный безопасный вариант. acks=1 → потеря при падении лидера до репликации. acks=0 → fire-and-forget
- Retries: default = MAX_INT (бесконечно). Контролировать через `delivery.timeout.ms`. Retry → возможны дубли → `enable.idempotence=true`
- Два типа ошибок: **retriable** (LEADER_NOT_AVAILABLE — retry поможет) и **non-retriable** (INVALID_CONFIG — retry бесполезен)
- Обязательно обрабатывать ошибки в коде: serialization errors, timeout, memory exhaustion — producer retries их НЕ ловят

---

## Два сценария потери данных

**Сценарий 1: acks=1, брокер настроен идеально** ^prod-scenario-acks1
```
1. Producer шлёт message → leader записал → ответ "success"
2. Leader падает ДО репликации на followers
3. Follower становится лидером — message НЕТ
4. Producer думает что message записан. Consumer его не увидит.
   Нет inconsistency (consumer не видел), но producer потерял данные.
```

**Сценарий 2: acks=all, нет обработки ошибок** ^prod-scenario-no-error-handling
```
1. Producer шлёт message → leader как раз упал
2. Kafka возвращает "Leader not Available"
3. Producer не обрабатывает ошибку, не делает retry
4. Message потерян — Kafka его вообще не получил
```

## Три режима acks

```
acks=0:  "отправил и забыл"
         ✗ не узнаем если партиция offline, кластер недоступен
         ✓ низкая latency на produce (но НЕ end-to-end)
         Консьюмеры всё равно ждут репликации

acks=1:  "лидер подтвердил"
         ✗ потеря при падении лидера до репликации
         ✗ можно записать быстрее чем реплицируется → under-replicated partitions
         ✓ баланс скорости и надёжности

acks=all: "все ISR подтвердили"
         ✓ самый надёжный (в связке с min.insync.replicas≥2)
         ✗ самая высокая produce latency
         ✓ producer ждёт полного commit перед следующим батчем
```
^prod-acks-comparison

**Формула надёжности:** `acks=all` + `min.insync.replicas=2` + `RF=3` = **золотой стандарт**. ^prod-golden-standard

## Retries

```
retries = MAX_INT (default, бесконечно)
delivery.timeout.ms = 120000 (default, 2 минуты)

→ Producer retry'ит retriable ошибки столько раз, сколько успеет
  за delivery.timeout.ms. Потом сдаётся.
```
^prod-retries-config

**Retriable errors:** LEADER_NOT_AVAILABLE, NOT_ENOUGH_REPLICAS, REQUEST_TIMED_OUT — retry может помочь ^prod-retriable

**Non-retriable errors:** INVALID_CONFIG, MESSAGE_TOO_LARGE, AUTHORIZATION — retry бесполезен, нужна обработка в коде ^prod-non-retriable

## Idempotent Producer

```
Проблема: retry → broker получил message дважды → дубли

Решение: enable.idempotence=true
→ producer добавляет sequence number в каждую запись
→ broker видит дубль → пропускает
→ at-least-once → exactly-once (на уровне одного producer)
```
^prod-idempotence

Подробности в [[Exactly-Once Semantics]] (Ch.8). ^prod-idempotence-ref

## Error Handling — что нужно обрабатывать в коде

Producer retries ловят только retriable broker errors. Остальное — на разработчике: ^prod-error-handling

```
Что producer retries НЕ обработают:
- Serialization errors (до отправки)
- Non-retriable broker errors (MESSAGE_TOO_LARGE, AUTH)
- Timeout (delivery.timeout.ms исчерпан)
- Memory exhaustion (буфер producer переполнен)

Что делать — зависит от бизнеса:
- Логировать и пропустить?
- Записать в dead-letter topic?
- Остановить чтение из источника (back pressure)?
- Сохранить на локальный диск?
```
^prod-error-handling-options

## Мониторинг producer

```
Метрики JMX:
- error-rate per record    — растёт → проблема с кластером
- retry-rate per record    — растёт → нестабильность

Логи (WARN level):
"Got error produce response... retrying (N attempts left)"
→ N=0 → retries исчерпаны, скоро потеря данных

ERROR level → message полностью потерян (non-retriable или timeout)
```
^prod-monitoring

## Связь
- [[Reliable Data Delivery — обзор]] — общие гарантии
- [[Broker Config — надёжность]] — min.insync.replicas + acks=all = golden standard
- [[Consumer — надёжность]] — consumer-side надёжность
- [[Producer — архитектура]] — producer flow, конфигурация, send methods (Ch.3)
- [[Partitioning]] — выбор партиции, ключи, стратегии (Ch.3)
- [[Idempotent Producer]] — дедупликация retry через PID + sequence (Ch.8)
- [[Transactions]] — atomic consume-process-produce (Ch.8)
- [[Delivery Semantics]] — at-most/at-least/exactly-once (Ch.8)
- [[Replication]] — ISR, как работает репликация (Ch.6)
- [[Request Processing]] — produce request flow, purgatory (Ch.6)

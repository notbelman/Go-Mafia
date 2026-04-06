- Даже при идеальной конфигурации брокеров, **producer может потерять данные** если неправильно настроен
- **acks=all** — единственный безопасный вариант. acks=1 → потеря при падении лидера до репликации. acks=0 → fire-and-forget
- Retries: default = MAX_INT (бесконечно). Контролировать через `delivery.timeout.ms`. Retry → возможны дубли → `enable.idempotence=true`
- Два типа ошибок: **retriable** (LEADER_NOT_AVAILABLE — retry поможет) и **non-retriable** (INVALID_CONFIG — retry бесполезен)
- Обязательно обрабатывать ошибки в коде: serialization errors, timeout, memory exhaustion — producer retries их НЕ ловят

---

## Два сценария потери данных

#### Сценарий 1: acks=1, брокер настроен идеально ^prod-scenario-acks1

1. Producer шлёт message → leader записал → ответ "success"
2. Leader падает **ДО** репликации на followers
3. Follower становится лидером — message НЕТ
4. Producer думает что message записан. Consumer его не увидит.

#### Сценарий 2: acks=all, нет обработки ошибок ^prod-scenario-no-error-handling

1. Producer шлёт message → leader как раз упал
2. Kafka возвращает "Leader not Available"
3. Producer не обрабатывает ошибку, не делает retry
4. Message потерян — Kafka его вообще не получил

## Три режима acks

| acks | Надёжность | Latency | Когда использовать |
|---|---|---|---|
| `0` | Нет гарантий | Минимальная | Метрики, логи (потеря допустима) |
| `1` | Лидер подтвердил | Средняя | Баланс скорости и надёжности |
| `all` | Все ISR подтвердили | Максимальная | Production данные |

> [!warning] acks=0
> Не узнаем если партиция offline или кластер недоступен. Консьюмеры всё равно ждут репликации — acks=0 снижает только produce latency.

> [!warning] acks=1
> Потеря при падении лидера до репликации. Можно записать быстрее чем реплицируется → under-replicated partitions.

^prod-acks-comparison

> [!tip] Золотой стандарт
> `acks=all` + `min.insync.replicas=2` + `RF=3` = максимальная надёжность. ^prod-golden-standard

## Retries

```text
retries = MAX_INT (default, бесконечно)
delivery.timeout.ms = 120000 (default, 2 минуты)

→ Producer retry'ит retriable ошибки столько раз, сколько успеет
  за delivery.timeout.ms. Потом сдаётся.
```

^prod-retries-config

| Тип ошибки | Примеры | Retry поможет? |
|---|---|---|
| **Retriable** | LEADER_NOT_AVAILABLE, NOT_ENOUGH_REPLICAS, REQUEST_TIMED_OUT | Да |
| **Non-retriable** | INVALID_CONFIG, MESSAGE_TOO_LARGE, AUTHORIZATION | Нет |

^prod-retriable
^prod-non-retriable

## Idempotent Producer

```text
Проблема: retry → broker получил message дважды → дубли

Решение: enable.idempotence=true
→ producer добавляет sequence number в каждую запись
→ broker видит дубль → пропускает
→ at-least-once → exactly-once (на уровне одного producer)
```

^prod-idempotence

Подробности в [[Idempotent Producer]] (Ch.8). ^prod-idempotence-ref

## Error Handling — что нужно обрабатывать в коде

Producer retries ловят только retriable broker errors. Остальное — на разработчике: ^prod-error-handling

**Что producer retries НЕ обработают:**
- Serialization errors (до отправки)
- Non-retriable broker errors (MESSAGE_TOO_LARGE, AUTH)
- Timeout (`delivery.timeout.ms` исчерпан)
- Memory exhaustion (буфер producer переполнен)

**Варианты реакции — зависит от бизнеса:**
- Логировать и пропустить?
- Записать в dead-letter topic?
- Остановить чтение из источника (back pressure)?
- Сохранить на локальный диск?

^prod-error-handling-options

## Мониторинг producer

| Метрика | Сигнал |
|---|---|
| `error-rate per record` | Растёт → проблема с кластером |
| `retry-rate per record` | Растёт → нестабильность |

**Логи (WARN level):**
```
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

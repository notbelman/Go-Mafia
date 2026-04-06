- Надёжность — свойство **системы**, не одного компонента. Брокеры, producers, consumers, сеть, диски — все участвуют
- Kafka гибкая: от "потерять можно" до "ни одного сообщения". Настройки позволяют выбрать trade-off
- Четыре гарантии Kafka: порядок в партиции, committed = на всех ISR, committed не теряется (пока жива хоть одна реплика), консьюмеры читают только committed
- **committed ≠ flushed to disk** — Kafka полагается на репликацию, не на fsync

---

## Четыре гарантии Kafka

1. **Порядок в партиции:** если B записано после A одним producer в одну партицию → offset B > offset A, консьюмер прочитает A раньше B ^rdd-guarantee-order

2. **Committed = записано на все ISR:** сообщение "committed" когда оно на всех in-sync репликах (не обязательно на диске) ^rdd-guarantee-committed

3. **Committed не теряется:** пока жива хотя бы одна реплика ^rdd-guarantee-durability

4. **Консьюмеры читают только committed:** сообщения до high-water mark ^rdd-guarantee-consumer

## Два значения "committed"

> [!warning] Не путать!
>
> | Термин | Что значит |
> |---|---|
> | **Committed message** | Сообщение записано на все ISR, доступно консьюмерам |
> | **Committed offset** | Консьюмер подтвердил: "я обработал всё до этого offset" |

^rdd-committed-vs-offset

## Trade-offs

Надёжность всегда за счёт чего-то: ^rdd-tradeoffs

| Параметр | Trade-off |
|---|---|
| **Availability** | Больше реплик → выше доступность, но больше ресурсов |
| **Throughput** | acks=all медленнее acks=1 |
| **Latency** | Ожидание репликации на все ISR добавляет задержку |
| **Disk space** | RF=3 = тройной объём данных |
| **Complexity** | Правильная обработка ошибок в producer/consumer |

## Карта главы

```text
            Reliable Data Delivery
                     │
     ┌───────────────┼───────────────┐
     ▼               ▼               ▼
  Broker Config   Producer        Consumer
  (RF, ISR,       (acks, retries, (offsets, auto-commit,
   unclean)        idempotence)    rebalance)
```

^rdd-map

## Связь
- [[Broker Config — надёжность]] — replication factor, min.insync.replicas, unclean election
- [[Producer — надёжность]] — acks, retries, error handling
- [[Consumer — надёжность]] — offset commit, rebalance, retry patterns
- [[Delivery Semantics]] — at-most/at-least/exactly-once (Ch.8)
- [[Replication]] — ISR, high-water mark (Ch.6)
- [[Kafka Internals — обзор]] — как устроен брокер изнутри (Ch.6)

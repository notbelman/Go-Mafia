- Chapter 6 книги. Четыре ключевых темы: **Controller**, **Replication**, **Request Processing**, **Physical Storage**
- Controller — один брокер, отвечает за выбор лидеров партиций. ZooKeeper-based (старый) → KRaft (новый, Raft-based)
- Replication — leader/follower реплики, ISR (in-sync replicas), high-water mark
- Request Processing — produce/fetch/admin запросы, zero-copy, purgatory
- Storage — сегменты, индексы, compaction, tiered storage

---

## Зачем знать internals

Не обязательно для работы с Kafka, но критично для: troubleshooting, тюнинг, собеседования. Понимание механизмов позволяет крутить ручки осознанно, а не наугад. ^internals-why

## Cluster Membership

Каждый брокер регистрируется в ZooKeeper (ephemeral node с broker ID). Если брокер падает — ephemeral node исчезает → все подписчики получают уведомление. ^internals-membership

Если запустить новый брокер с тем же ID что у упавшего — он займёт его место: получит те же партиции и реплики. ^internals-broker-replace

## Карта главы

```
                    Kafka Internals
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   Controller       Replication      Request Processing
   (ZK + KRaft)     (ISR, HW)       (Produce, Fetch)
                                          │
                                          ▼
                                   Physical Storage
                                   (Segments, Index,
                                    Compaction)
```
^internals-map

## Связь
- [[Controller]] — выбор лидеров партиций (ZooKeeper)
- [[KRaft]] — новый контроллер на базе Raft
- [[Replication]] — leader/follower, ISR, high-water mark
- [[Request Processing]] — как брокер обрабатывает запросы
- [[Storage — сегменты и файлы]] — физическое хранение данных
- [[Compaction]] — log compaction и tombstones

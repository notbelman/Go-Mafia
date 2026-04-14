- **Replication Factor (RF):** RF=N → можно потерять N-1 брокеров. RF=3 — стандарт. Trade-off: availability vs disk space и inter-broker traffic
- **min.insync.replicas:** минимум ISR для записи. RF=3 + min.insync=2 + acks=all — **золотой стандарт** надёжности
- **unclean.leader.election.enable=false** (default): out-of-sync реплика НЕ может стать лидером. Безопаснее, но при потере всех ISR — партиция offline
- Kafka **НЕ ждёт fsync** — пишет в page cache, полагается на репликацию. Можно настроить flush.messages / flush.ms, но это бьёт по throughput

---

## Replication Factor

| RF | Поведение |
|---|---|
| **RF=1** | Один брокер упал → данные потеряны, партиция offline |
| **RF=2** | Один брокер упал → всё ок, inter-broker traffic x1 |
| **RF=3** | Два брокера упали → всё ок, inter-broker traffic x2 |

^bc-rf-examples

**Пять факторов выбора RF:** ^bc-rf-factors
- **Availability:** RF=1 → даже плановый рестарт = downtime
- **Durability:** больше копий → ниже вероятность потери всех
- **Throughput:** каждая дополнительная реплика = дополнительный inter-broker traffic. 10 MBps produce + RF=3 → 20 MBps replication traffic
- **Latency:** больше реплик → выше вероятность что одна будет медленной
- **Cost:** RF=2 вместо 3 для некритичных данных. Если storage уже реплицирует (RAID/cloud) — durability обеспечена, но availability ниже

> [!tip] Rack awareness
> Реплики одной партиции на **разных стойках** (`broker.rack`). В облаке = разные AZ. Иначе одна стойка упала → все реплики потеряны, RF бессмыслен. ^bc-rack-awareness

## min.insync.replicas

```text
RF=3, min.insync.replicas=2, acks=all:

3 ISR: ✓ запись работает, все 3 получают
2 ISR: ✓ запись работает, обе получают
1 ISR: ✗ NotEnoughReplicasException — запись отклонена
         но чтение работает (read-only mode)
0 ISR: ✗ партиция offline
```

^bc-min-isr-scenario

> [!important]
> Без min.insync.replicas, даже с acks=all, если ISR сократился до 1 реплики — "all" = 1, и данные могут быть потеряны при падении этой реплики. `min.insync.replicas=2` защищает от этого. ^bc-min-isr-why

**Восстановление из read-only:** вернуть один из недоступных брокеров → дождаться пока реплика догонит лидер и войдёт в ISR → запись снова работает. ^bc-min-isr-recovery

## Unclean Leader Election

```text
Сценарий: RF=3, два follower упали, leader единственный ISR.
Producer пишет offsets 100-200 на leader.
Leader тоже упал. Follower 0 поднялся (у него только 0-100).
```

| Режим | Что происходит |
|---|---|
| **unclean=true** | Follower 0 становится лидером. Offsets 100-200 потеряны. Данные inconsistent. |
| **unclean=false** | Партиция OFFLINE пока старый leader не вернётся. Данные не потеряны, но нет availability. |

^bc-unclean-scenario

> [!note]
> Default = `false` (безопаснее). Админ может временно включить `true` для восстановления availability, но потом вернуть `false`. ^bc-unclean-default

## Keeping Replicas In Sync

|Параметр|Default (2.5.0+)|Что контролирует|
|:--|:--|:--|
|`zookeeper.session.timeout.ms`|18s (было 6s)|Как долго брокер может не слать heartbeat в ZK|
|`replica.lag.time.max.ms`|30s (было 10s)|Как долго реплика может отставать от лидера|

^bc-keeping-sync

> [!warning]
> **Побочный эффект** `replica.lag.time.max.ms=30s`: консьюмер может ждать до 30 секунд пока сообщение станет committed (реплицируется на все ISR). ^bc-lag-latency-impact

## Persistence to Disk

```text
Kafka writes → Linux page cache → OS flushes when full
                                   + при ротации сегментов (1GB)
                                   + при рестарте
```

^bc-persistence

**Идея:** 3 машины в разных стойках/AZ, каждая с копией данных — надёжнее чем fsync на одной машине. Одновременный отказ двух стоек/AZ крайне маловероятен. ^bc-persistence-reasoning

Можно настроить `flush.messages` и `flush.ms` для более частого fsync, но это **сильно бьёт по throughput**. ^bc-flush-config

## Связь
- [[Reliable Data Delivery — обзор]] — общие гарантии и trade-offs
- [[Producer — надёжность]] — acks взаимодействует с min.insync.replicas
- [[Replication]] — ISR, high-water mark, leader election (Ch.6)
- [[Controller]] — контроллер выполняет leader election (Ch.6)
- [[Storage — сегменты и файлы]] — сегменты, page cache, fsync (Ch.6)

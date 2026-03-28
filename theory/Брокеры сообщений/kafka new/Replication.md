- Replication — основа отказоустойчивости Kafka. Каждая партиция имеет **leader** + **follower** реплики
- **Leader** — принимает все produce запросы. **Followers** — реплицируют данные с лидера через Fetch запросы
- **ISR (In-Sync Replicas)** — реплики, которые не отстали от лидера. Только ISR могут стать новым лидером
- **High-Water Mark (HW)** — offset последнего сообщения, реплицированного на ВСЕ ISR. Консьюмеры читают только до HW
- Реплика out of sync: не запрашивала данные >10s ИЛИ запрашивает, но не догнала лидер за >10s (`replica.lag.time.max.ms`)

---

## Два типа реплик

**Leader replica** — одна на партицию. Все produce запросы идут через лидера (гарантия consistency). Клиенты могут читать и с лидера, и с фолловеров. ^repl-leader

**Follower replica** — все остальные реплики. Основная задача: реплицировать сообщения с лидера и оставаться в sync. При падении лидера — один из followers становится новым лидером. ^repl-follower

## ISR — In-Sync Replicas

```
Партиция 0, replication factor = 3:

  Broker 1: Leader    [msg1 msg2 msg3 msg4 msg5]   ← produce сюда
  Broker 2: Follower  [msg1 msg2 msg3 msg4 msg5]   ← in sync ✓
  Broker 3: Follower  [msg1 msg2 msg3]              ← отстал ✗

  ISR = {Broker 1, Broker 2}
  High-Water Mark = offset msg5 (реплицировано на все ISR)
```
^repl-isr-example

Followers шлют лидеру **Fetch запросы** (такие же, как консьюмеры). По последнему запрошенному offset лидер знает, насколько каждая реплика отстала. ^repl-fetch-mechanism

**Out of sync:** реплика не запрашивала данные >10 секунд, ИЛИ запрашивает но не догнала лидер за >10 секунд. Контролируется `replica.lag.time.max.ms`. ^repl-out-of-sync

**Только ISR** могут быть избраны лидером при failover — они содержат все подтверждённые сообщения. ^repl-isr-leader-election

## High-Water Mark (HW)

Консьюмеры читают **только до HW** — offset последнего сообщения, записанного на все ISR. ^repl-hw-def

```
Leader:  [msg1 msg2 msg3 msg4 msg5 msg6]
                              ▲         ▲
                              HW        LEO (Log End Offset)

Сообщения msg5, msg6 ещё не на всех ISR →
консьюмеры их не видят (получат пустой ответ, не ошибку)
```
^repl-hw-visibility

**Зачем:** если лидер упадёт и msg5/msg6 потеряются — консьюмер, который их уже прочитал, увидит inconsistency. Поэтому ждём репликацию на все ISR. ^repl-hw-why

**Побочный эффект:** медленная репликация → задержка для консьюмеров. Задержка ограничена `replica.lag.time.max.ms`. ^repl-hw-delay

## Read from Follower (KIP-392)

С KIP-392 консьюмеры могут читать с **ближайшей in-sync реплики** вместо лидера. Цель — снизить сетевые расходы (кросс-датацентр). ^repl-read-follower

```
consumer config:  client.rack = "rack-A"
broker config:    replica.selector.class = RackAwareReplicaSelector
                  rack.id = "rack-A"
```
^repl-read-follower-config

Гарантии те же: читаются только committed сообщения (до HW). Но HW на фолловере обновляется с небольшой задержкой → данные на лидере доступны раньше. ^repl-read-follower-delay

## Preferred Leader

Preferred leader = реплика, которая была лидером при создании топика. Всегда **первая в списке реплик**. ^repl-preferred-leader

Зачем: при создании партиций лидеры равномерно распределены по брокерам. После failover/failback баланс может нарушиться. `auto.leader.rebalance.enable=true` → Kafka автоматически возвращает preferred leader, если он в ISR. ^repl-preferred-rebalance

## Связь
- [[Controller]] — контроллер выбирает лидеров при failover
- [[KRaft]] — KRaft управляет ISR и лидерами
- [[Request Processing]] — produce/fetch запросы и acks
- [[Kafka Internals — обзор]] — место репликации в архитектуре
- [[Broker Config — надёжность]] — RF, min.insync.replicas, unclean election (Ch.7)
- [[Producer — надёжность]] — acks=all + ISR = гарантия commit (Ch.7)
- [[Idempotent Producer]] — follower хранит producer state (PID/seq) при репликации (Ch.8)

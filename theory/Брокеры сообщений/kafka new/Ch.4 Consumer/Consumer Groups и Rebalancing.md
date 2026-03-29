- Consumer group: каждый consumer получает **subset партиций**. Масштабирование = добавить consumer'ов. Больше consumer'ов чем партиций → лишние idle
- Разные приложения = **разные group.id** → каждое получает ВСЕ сообщения. Один group.id = делят партиции между собой
- **Eager rebalance:** все consumer'ы останавливаются, отдают партиции, получают заново. STW-пауза
- **Cooperative rebalance:** только subset партиций перераспределяется, остальные продолжают работать. Несколько фаз, но без полной остановки
- **Static membership** (`group.instance.id`): consumer при рестарте получает **те же партиции** без rebalance. Для stateful consumers с локальным кэшем

---

## Consumer Groups — масштабирование

```
Topic T1 (4 partitions):

1 consumer:   C1 ← P0, P1, P2, P3        (всё на одном)
2 consumers:  C1 ← P0, P1  C2 ← P2, P3   (поровну)
4 consumers:  C1←P0  C2←P1  C3←P2  C4←P3  (по одной)
5 consumers:  C1←P0  C2←P1  C3←P2  C4←P3  C5←idle!
```
^cg-scaling

**Правило:** максимум полезных consumer'ов = количество партиций. Поэтому создавай топики с запасом партиций. ^cg-max-consumers

**Разные приложения:** разный `group.id` → каждая группа получает все сообщения независимо. Kafka масштабируется на большое число групп без потери производительности. ^cg-multiple-groups

## Rebalance — перераспределение партиций

Когда происходит: consumer добавлен/удалён/крашнулся, партиции добавлены, подписка изменилась. ^cg-rebalance-triggers

### Eager Rebalance

```
Фаза 1: ВСЕ consumer'ы отдают ВСЕ партиции
         → полная остановка потребления (STW)
Фаза 2: Все заново присоединяются к группе
         → получают новое распределение
         → возобновляют потребление

Проблема: даже если перераспределяется 1 партиция — останавливаются ВСЕ
```
^cg-eager

### Cooperative (Incremental) Rebalance

```
Фаза 1: Group leader говорит: "C1, отдай P2"
         → C1 останавливает ТОЛЬКО P2
         → C2, C3 продолжают работать
Фаза 2: P2 назначается C4
         → C4 начинает потреблять P2

Может занять несколько итераций, но нет полной остановки
```
^cg-cooperative

## Heartbeats и обнаружение сбоев

```
Consumer → Group Coordinator (фоновый поток шлёт heartbeats)

session.timeout.ms = 10s (default)
  → нет heartbeat 10s → consumer считается мёртвым → rebalance

heartbeat.interval.ms = 3s (default, обычно 1/3 от session.timeout)
  → как часто шлём heartbeat

max.poll.interval.ms = 5 min (default)
  → нет вызова poll() за 5 мин → consumer мёртв
  → защита от: main thread завис, но heartbeat thread жив
```
^cg-heartbeats

**Два механизма обнаружения:** heartbeat (сетевой сбой, крэш) + poll interval (зависание main thread). ^cg-two-detection

## Assignment Strategies

```
partition.assignment.strategy:

Range (default):
  C1, C2 подписаны на T1(3 part) и T2(3 part)
  C1 ← T1-P0,P1 + T2-P0,P1    (по 2 от каждого)
  C2 ← T1-P2 + T2-P2           (по 1 от каждого)
  → C1 получает больше (нечётное деление)

RoundRobin:
  Все партиции всех топиков по очереди
  → более равномерно чем Range

Sticky:
  Как RoundRobin, но при rebalance — максимально сохраняет текущее распределение
  → меньше перемещений партиций

CooperativeStickyAssignor:
  Как Sticky + поддержка cooperative rebalance
  → рекомендуется для production
```
^cg-assignment-strategies

## Static Group Membership

```
// Consumer с фиксированной identity:
props.put("group.instance.id", "consumer-host-1");

Обычный consumer:
  shutdown → leave group → rebalance → restart → новые партиции

Static member:
  shutdown → НЕ покидает группу (до session.timeout.ms)
  restart  → получает ТЕ ЖЕ партиции без rebalance
```
^cg-static-membership

**Use case:** consumer с локальным state/кэшем, который дорого пересоздавать. При рестарте — тот же state, те же партиции. ^cg-static-usecase

**Trade-off:** пока consumer offline, его партиции не обрабатываются. `session.timeout.ms` должен быть: достаточно высоким для рестартов, достаточно низким для реальных сбоев. ^cg-static-tradeoff

## Как работает назначение партиций внутри

```
1. Consumer шлёт JoinGroup → Group Coordinator
2. Первый consumer = Group Leader
3. Leader получает список всех consumer'ов
4. Leader назначает партиции (PartitionAssignor)
5. Leader шлёт назначения → Coordinator → все consumer'ы
6. Каждый consumer видит только СВОИ партиции
```
^cg-assignment-internals

## Связь
- [[Consumer — надёжность]] — offset commit, retry patterns (Ch.7)
- [[Consumer — архитектура]] — poll loop, конфиги, seek (Ch.4)
- [[Partitioning]] — количество партиций определяет макс. параллелизм (Ch.3)
- [[Replication]] — consumer читает до high-water mark (Ch.6)
- [[Transactions]] — isolation.level, read_committed (Ch.8)

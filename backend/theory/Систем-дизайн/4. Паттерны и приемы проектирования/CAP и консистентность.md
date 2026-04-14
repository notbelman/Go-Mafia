- **CAP-теорема:** при сетевом разделении (P) можно гарантировать только **C** (консистентность) или **A** (доступность). P — не опция, сеть **будет** рваться
- **CP** (PostgreSQL, MongoDB, etcd) — при partition отклоняем запросы, пока не восстановим консистентность. Данные правильные, но может быть недоступна
- **AP** (Cassandra, Elasticsearch, DynamoDB) — при partition отвечаем, но данные могут быть устаревшими. Доступна, но eventual consistency
- **PACELC** расширяет CAP: когда сеть **ок** — выбираем **L** (скорость, ответ с ближайшего узла) или **C** (правильность, ждём кворум). Cassandra = PA/EL, PostgreSQL = PC/EC
- **Strong consistency** — после записи любое чтение с любого узла = свежее значение. Цена: [[Производительность|latency]] (ждём кворум)
- **Eventual consistency** — узлы синхронизируются когда-нибудь (мс–сек). Конфликты: LWW, vector clocks, CRDTs
- **Tunable** (Cassandra): ONE = быстро/eventual, QUORUM = ближе к strong, ALL = strong но одна упавшая нода = недоступность. Формула: **W + R > N → strong**
- CAP — не постоянная метка. Cassandra с QUORUM ведёт себя как CP. DynamoDB с consistent read — тоже

---

## Сравнение

| БД | CAP | PACELC | Почему |
|---|---|---|---|
| PostgreSQL | CP | PC/EC | Single-leader, strong consistency всегда |
| MySQL | CP | PC/EC | Single-leader, strong для мастера |
| MongoDB | CP | PC/EC | Replica set с выборами лидера |
| etcd / Consul | CP | PC/EC | Raft consensus, без кворума — read-only |
| Cassandra | AP | PA/EL | Masterless, eventual по умолчанию |
| Elasticsearch | AP | PA/EL | Eventual (refresh interval 1s) |
| DynamoDB | AP | PA/EL | Eventual по умолчанию, strong опционально |

## CAP-теорема — суть

### Три свойства
- **C — Consistency:** прочитал после записи — свежее значение, с любого узла
- **A — Availability:** каждый запрос получает ответ (не ошибку), даже если часть узлов упала
- **P — Partition Tolerance:** система работает при потере связи между узлами

### Реальный выбор
P — не опция. В распределённой системе сеть **будет** рваться. Поэтому выбор: **CP** (правильно, но может быть недоступно) или **AP** (доступно, но может врать).

## PACELC — расширение CAP

CAP описывает поведение **только при partition**. Но это редкость — 99.9% времени всё работает. PACELC добавляет второй вопрос:

| Режим | Вопрос | Варианты |
|---|---|---|
| **PAC** (сеть порвалась) | Что выбираем? | **A** (отвечаем) или **C** (отклоняем) |
| **ELC** (сеть ок) | Что выбираем? | **L** (быстро, с ближайшего) или **C** (правильно, ждём кворум) |

**Почему полезнее CAP:** Cassandra жертвует консистентностью **даже когда всё хорошо** ради скорости. PostgreSQL консистентен **всегда**.

## Strong vs Eventual Consistency

### Strong
После записи **любое** чтение с **любого** узла = свежее значение. Достигается: кворум, single-leader, consensus (Raft, Paxos). Цена: [[Производительность|latency]].

### Eventual
Узлы синхронизируются **когда-нибудь** (мс–сек). Конфликты: last-write-wins (LWW), vector clocks, CRDTs. Цена: stale reads, возможная потеря данных при LWW.

### Tunable Consistency (Cassandra)

| Уровень | Что значит |
|---|---|
| ONE | Ответ от одной ноды — быстро, eventual |
| QUORUM | Большинство подтвердило — ближе к strong |
| ALL | Все подтвердили — strong, но одна упавшая = недоступность |

**Формула:** W + R > N → strong consistency.

## Связь
- [[Выбор базы данных]] — CAP влияет на выбор БД
- [[BASE и блокировки транзакций]] — BASE = eventual consistency по умолчанию
- [[Виды баз данных]] — реляционные = CP, NoSQL = часто AP

## Быстрая навигация

- [[Менторство/моки/PostgreSQL & SQL/общее]] — что такое Redis, зачем, vs Memcached
- [[Кеширование]] — cache miss, почему быстрый, multi-layer
- [[Инвалидация кеша]] — TTL, event-based, pub/sub, версионирование
- [[Проблемы кеширования]] — avalanche, stampede, penetration, cold start
- [[Персистентность - RDB vs AOF]] — RDB снепшоты vs AOF журнал
- [[Sentinel]] — репликация, failover, автовыбор мастера
- [[Split Brain]] — проблемы распределённых систем, потери данных
- [[Масштабирование]] — Redis Cluster, хеш-слоты
- [[Падение Redis]] — circuit breaker, fallback, graceful degradation

---

## [[Менторство/моки/PostgreSQL & SQL/общее]]

#### [[Для чего чаще всего используется]]
Кэш, сессии, rate limiting, pub/sub, distributed locks, leaderboards

#### [[В чем основные отличия от обычной СУБД]]
In-memory, структуры данных, single-threaded, persistence опциональна

#### [[Redis vs Memcached]]
Когда Redis, когда Memcached — типы данных, persistence, cluster, pub/sub

---

## [[Кеширование]]

#### [[Зачем используют кеширование]]
Снижение latency, разгрузка БД, масштабирование read-heavy нагрузки

#### [[Почему Redis быстрый (Performance)]]
In-memory, single-threaded event loop, O(1) операции, zero system calls

#### [[Что происходит, если данных нет в кеше (Cache Miss)]]
Cache-aside паттерн, thundering herd, cache warming

#### [[Эшелонированные кэши (Multi-Layer Cache)]]
L1 (in-process) → L2 (Redis) → L3 (DB), trade-offs каждого уровня

---

## [[Инвалидация кеша]]

#### [[Инвалидация кэша]]
Обзор стратегий инвалидации, trade-offs

#### [[TTL (Time-To-Live)]]
Простейшая стратегия, stale window, как выбрать TTL

#### [[Event-based (Delete on Write)]]
Удаление при записи в БД, cache-aside + delete on write

#### [[Pub - Sub (через Redis, Kafka, NATS)]]
Событийная инвалидация через брокер, fan-out на несколько инстансов кэша

#### [[Версионирование ключа]]
Версия в ключе, автоматическая инвалидация без явного удаления

---

## [[Проблемы кеширования]]

#### [[Проблемы кэширования]]
Обзор всех проблем

#### [[Cache Stampede (Thundering Herd)]]
Одновременный пересчёт при истечении TTL — mutex lock, probabilistic early expiry

#### [[Cache Avalanche]]
Массовое истечение TTL одновременно — jitter, разброс TTL

#### [[Cache Penetration]]
Запросы несуществующих ключей бьют в БД — bloom filter, null caching

#### [[Stale Data (устаревшие данные)]]
Устаревшие данные в кэше — stale-while-revalidate, read-through

#### [[Cold Start (холодный кэш)]]
Пустой кэш после перезапуска — cache warming, gradual rollout

---

## [[Персистентность - RDB vs AOF]]

#### [[RDB (Redis Database) — Снепшоты]]
BGSAVE, fork + copy-on-write, периодические снепшоты, быстрый старт

#### [[AOF (Append-Only File) — Журналирование]]
Запись каждой команды, fsync стратегии (always/everysec/no), AOF rewrite

#### [[Сравнение механизмов]]
RDB vs AOF vs RDB+AOF — durability, recovery time, размер, производительность

---

## [[Sentinel]]

#### [[Что это]]
Redis Sentinel — мониторинг, нотификации, автоматический failover

#### [[Механизм репликации]]
Master-replica асинхронная репликация, REPL_ID, offset, partial resync

#### [[Redis Sentinel]]
Кворум, объективный down vs субъективный, выбор нового мастера

---

## [[Split Brain]]

#### [[Что это]]
Split Brain в Redis — два мастера одновременно после network partition

#### [[Феномен Split Brain (Разделение мозга)]]
Как возникает, почему опасен, потери данных

#### [[Потери данных при асинхронной репликации]]
Replication lag, данные принятые мастером но не дошедшие до реплики

#### [[Как минимизировать риски (Mitigation)]]
min-replicas-to-write, min-replicas-max-lag, wait команда

---

## [[Масштабирование]]

#### [[Что это]]
Redis Cluster — горизонтальное масштабирование, sharding на уровне Redis

#### [[Архитектура и Хеш-слоты]]
16384 хеш-слота, CRC16 % 16384, resharding, MOVED редирект

#### [[Преимущества кластера]]
Линейное масштабирование, автоматический failover без Sentinel

---

## [[Падение Redis]]

#### [[Что делать при падении Redis]]
Стратегии: fallback на БД, degraded mode, circuit breaker

#### [[Fallback на БД]]
Прямые запросы в БД при недоступности Redis, кэш-прогрев после восстановления

#### [[Circuit Breaker]]
Паттерн автоматического отключения нагрузки при ошибках Redis

#### [[Graceful Degradation]]
Плавная деградация — отключение некритичных фич при падении кэша

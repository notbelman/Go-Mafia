# Моя часть — Tracking Service

## Контекст

Tracking Service — потребитель `telematics.gps` из телематического pipeline. Получает нормализованные GPS-события от Telematics Service и делает три вещи:

1. **Write** — сохранить текущую позицию (Redis) и историю маршрута (PostgreSQL)
2. **Read** — отдать текущую позицию или историю по запросу диспетчера
3. **Serve** — REST API для фронтенда: одна позиция, пачка позиций, история маршрута (polling-friendly)

```
telematics.gps (Kafka, ~1 500 events/sec)
       │
       ▼
Tracking Service (consumer group, 2-4 инстанса, полностью stateless)
       │
       ├──→ Redis HSET vehicle:{id}          (текущая позиция, hot)
       │
       ├──→ Buffer → PG batch INSERT          (история маршрута, warm)
       │
       └──→ REST API                          (GET /positions, Redis HGET/MGET)
```

---

## Нагрузка

### Входные данные

| Параметр | Значение |
|---|---|
| Источник | Kafka топик `telematics.gps` |
| RPS на входе | ~1 500 events/sec |
| Активных ТС | 15 000 |
| Частота GPS | каждые 10 сек (премиум-тариф провайдеров) |
| Формат | Protobuf (`RawTelematicsEvent`, type = GPS) |
| Retention истории | 3 года (требование заказчика, compliance) |

### Write path

| Операция | RPS | Куда |
|---|---|---|
| Обновление текущей позиции | 1 500 HSET/sec | Redis |
| Запись в историю | 1 500 rows/sec | PostgreSQL (через batch INSERT) |

### Read path

| Операция | RPS | Откуда | Кто |
|---|---|---|---|
| `GET /tracking/vehicles/{id}/current` | ~50 req/sec | Redis `HGETALL` → O(1) | Карточка ТС, звонок от клиента |
| `GET /tracking/positions?vehicles=...` | ~250 req/sec | Redis pipeline `HGETALL` | Карта диспетчера (poll каждые 2 сек × 500 диспетчеров) |
| `GET /tracking/vehicles/{id}/history?from=&to=` | ~50 req/sec | PG → partition pruning + index scan | Диспетчеры, отчёты |

**Соотношение write/read: 1 500 / 350 ≈ 4:1.** Write-heavy сервис, но в разумных пропорциях.

### Storage — расчёт

```
Размер одной строки в PG:
  vehicle_id    UUID          16 байт
  timestamp     BIGINT         8 байт
  lat           DOUBLE          8 байт
  lon           DOUBLE          8 байт
  speed         DOUBLE          8 байт
  heading       DOUBLE          8 байт
  altitude      DOUBLE          8 байт
  satellites    INTEGER         4 байта
  hdop          DOUBLE          8 байт
  provider      VARCHAR(20)   ~16 байт
  ─────────────────────────────────────
  Итого row:                  ~92 байта

  + tuple header PG (~23 байта)
  + alignment padding (~5 байт)
  + index (vehicle_id, timestamp) B-tree: ~50 байт/строка
  ─────────────────────────────────────
  Итого с overhead:           ~170 байт/строка
```

```
Per second:  1 500 × 170 B = 255 KB/sec
Per day:     0.255 × 86 400 = 22 GB/day
Per month:   22 × 30        = 660 GB/month
Per year:    660 × 12       = 7.9 TB/year
Per 3 years:                = ~24 TB (полный retention)
```

**24 ТБ за 3 года — это ещё управляемо в PostgreSQL, но держать всё в одном инстансе неразумно:** VACUUM становится долгим, бэкапы по часу, индексы разбухают, partition pruning начинает тормозить на сотнях партиций. Плюс 95% запросов диспетчеров — за последние 24-48 часов. Отсюда hot/warm/cold stratification.

---

## Hot / Warm / Cold — трёхуровневое хранение

### Зачем три уровня

24 ТБ в одном PG — терпимо по объёму, но болезненно операционно: VACUUM часами, бэкапы долгие, index bloat, партиции без boundary planning начинают тормозить. Плюс 95% запросов диспетчеров — за последние 24-48 часов. Старые данные нужны только для compliance.

### Три уровня

```
┌──────────────────────────────────────────────────────────────┐
│ HOT: Redis                                                   │
│ Текущая позиция каждого ТС                                   │
│ 15 000 ключей × 200 байт ≈ 3 МБ                              │
│ Запросы: "Где truck-42 сейчас?" → HGETALL → O(1), <1 мс      │
│ TTL: 5 мин (dead man switch — нет данных 5 мин = "офлайн")   │
└──────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│ WARM: PostgreSQL                                             │
│ История за последние 30 дней                                 │
│ ~660 ГБ, ежемесячные партиции                                │
│ Запросы: "Маршрут truck-42 за вчера" → index scan, <50 мс    │
│ Индекс: (vehicle_id, timestamp) → partition pruning          │
└──────────────────────────────────────────────────────────────┘
                         │
              архивация (cron, 1 раз/месяц)
                         ▼
┌──────────────────────────────────────────────────────────────┐
│ COLD: S3 (MinIO)                                             │
│ Архив: 30+ дней, compressed pg_dump                          │
│ ~24 ТБ за 3 года (с compression ~2-3x → ~8-12 ТБ)            │
│ Запросы: почти никогда. Compliance, legal, инциденты         │
│ Восстановление: ATTACH партиции обратно через pg_restore     │
└──────────────────────────────────────────────────────────────┘
```

### Почему 30 дней warm, не 90

| Вариант | Объём в PG | Проблемы |
|---|---|---|
| 7 дней | ~150 ГБ | Мало — ежемесячные отчёты не покрываются |
| **30 дней** | **~660 ГБ** | **Покрывает 95% запросов + ежемесячные отчёты** |
| 90 дней | ~2 ТБ | Ещё терпимо по объёму, но VACUUM/бэкапы тяжелее, index bloat заметнее |
| 365 дней | ~8 ТБ | Операционно тяжело: долгие бэкапы, затяжной VACUUM, медленные DDL |

30 дней — sweet spot: покрывает операционные нужды, PG справляется без боли.

### Процесс архивации

```
Ежемесячный cron (1-е число, 03:00):

1. DETACH partition:
   ALTER TABLE position_history DETACH PARTITION position_history_2026_01;

2. Dump + compress:
   pg_dump -t position_history_2026_01 | gzip > 2026_01.sql.gz

3. Upload в S3:
   mc cp 2026_01.sql.gz minio/tracking-archive/2026/01/

4. Verify (checksum):
   md5 локального файла == md5 в S3

5. Drop:
   DROP TABLE position_history_2026_01;

6. Метрика: archive_partitions_total{status="success"}
```

**Если S3 недоступен:** cron откладывает, не удаляет партицию. Retry через 1 час. Алерт если не удалось за 24 часа.

**Если нужны старые данные (восстановление из cold storage):**

Сценарий: аналитик или юрист хочет данные за январь 2026 (партиция уже в S3).

```
1. Запрос через тикет в Jira (не самообслуживание — DBA контролирует)

2. DBA восстанавливает партицию:
   mc cp minio/tracking-archive/2026/01/2026_01.sql.gz .
   gunzip 2026_01.sql.gz | psql → создаёт таблицу position_history_2026_01

3. ATTACH обратно как партицию (read-only, на реплике):
   ALTER TABLE position_history ATTACH PARTITION position_history_2026_01
   FOR VALUES FROM (...) TO (...);

4. Партиция доступна для запросов — все пользователи видят данные
   через обычные SELECT, PG сам делает partition pruning

5. Через 7 дней (или по завершении задачи):
   DETACH + DROP. Или оставить если данные нужны регулярно.
```

**Почему ATTACH, а не temp-таблица:**
- 10 аналитиков хотят тот же январь → одно восстановление, все видят
- Temp-таблица = per-session, не расшарена. Каждый аналитик = отдельное восстановление одного и того же файла
- ATTACH = партиция видна всем, обычные запросы работают без изменений
- Partition pruning работает: запрос за февраль не трогает восстановленный январь

**Время восстановления:**
- Скачивание из S3: ~1-5 минут (660 ГБ / партиция, compressed ~200-300 ГБ)
- pg_restore: ~10-30 минут
- Итого: **~15-45 минут.** Для compliance/legal — допустимо.

**Автоматизация (если восстановления частые):**
API-эндпоинт: `POST /admin/archive/restore?month=2026-01` → фоновый job → Kafka event → Slack notification когда готово. Но у нас такого не было — восстановления были ~1 раз в квартал.

---

## Выбор БД — почему PostgreSQL, а не ClickHouse

### Контекст

1.5K write/sec и 24 ТБ за 3 года — объёмы, при которых ClickHouse можно рассмотреть. Мы обсуждали оба варианта.

### Сравнение

| Критерий | PostgreSQL | ClickHouse |
|---|---|---|
| **Тип запросов Tracking** | Point query: "маршрут truck-42, 10:00-12:00 сегодня" | Scan query: "средняя скорость всех ТС за квартал" |
| **Write** | Batch INSERT 1.5K/sec — легко | Native batch — тоже легко |
| **Point query latency** | <50 мс (B-tree index на (vehicle_id, timestamp)) | ~200-500 мс (sparse index, full block scan) |
| **Compression** | 2-3x | **10-50x** (24 ТБ → 2-5 ТБ) |
| **Storage за 3 года** | ~8-12 ТБ (с compression + archival в S3) | ~2-5 ТБ (всё в ClickHouse) |
| **UPDATE/DELETE** | Нормально | Дорого (ReplacingMergeTree) |
| **ACID** | Да | Нет (eventual merge) |
| **Operational cost** | Команда знает PG, Patroni есть | **Новая технология** — обучение, другой operational model |
| **Экосистема** | pg_partman, Patroni, pgBackRest | Своя репликация (ReplicatedMergeTree) |

### Почему PG победил

**1. Паттерн запросов Tracking — point queries.**

Диспетчер хочет: "покажи маршрут truck-42 за последние 2 часа". Это `WHERE vehicle_id = X AND timestamp BETWEEN A AND B` — классический B-tree index scan. PG делает это за миллисекунды.

ClickHouse оптимизирован под full-column scans: "посчитай среднюю скорость по всем 15K ТС за месяц". Это задача Analytics Service, не Tracking.

**2. Команда знает PG.**

Четыре бэкендера + DBA заказчика. Все работали с PG. ClickHouse никто не знал. Добавлять новую технологию = обучение + новый operational pipeline + новые runbook-и для дежурных. Для одного сервиса — оверкилл.

**3. ClickHouse уже есть — в Analytics Service.**

Analytics Service (зона ответственности Максима) использует ClickHouse для OLAP: агрегации, дашборды, отчёты. Данные туда приходят из Kafka (`telematics.engine`, `telematics.gps`). То есть ClickHouse в системе есть, но каждый сервис использует хранилище под свои задачи.

**4. Storage решается hot/warm/cold без героики.**

24 ТБ "в лоб" в PG — болезненно, но с archival: 660 ГБ warm + S3 cold. Управляемо без экзотики.

### Trade-off (честно для интервью)

"PG vs ClickHouse был обсуждаемым решением. Мы выбрали PG потому что (а) Tracking делает point queries, для которых B-tree эффективнее sparse index, (б) команда знает PG, ClickHouse никто не тянул operationally, (в) разница в storage невелика на текущих объёмах — 8-12 ТБ в PG+S3 vs 2-5 ТБ в ClickHouse не стоит переучивания команды. Если бы аналитические запросы по исторической GPS-истории стали частыми — ClickHouse был бы следующим шагом. Аналитические сценарии мы отдали в Analytics Service (тоже на ClickHouse, но это зона Максима)."

---

## PostgreSQL — детали

### Batch INSERT

**Проблема:** 1500 individual INSERT/sec = 1500 транзакций = 1500 WAL flush-ей в секунду. PostgreSQL деградирует: WAL writer становится bottleneck, latency вырастает до десятков миллисекунд.

**Решение: буфер + batch INSERT через `pgx.Batch`.**

```go
// В каждом инстансе Tracking Service:
buffer := make([]PositionEvent, 0, 300)

for event := range kafkaConsumer {
    buffer = append(buffer, event)

    if len(buffer) >= 300 || timeSinceLastFlush > 200*time.Millisecond {
        pgBatchInsert(buffer)
        buffer = buffer[:0]
        commitKafkaOffset()
    }
}

func pgBatchInsert(events []PositionEvent) {
    batch := &pgx.Batch{}
    for _, e := range events {
        batch.Queue(
            `INSERT INTO position_history
             (vehicle_id, timestamp, lat, lon, speed, heading, altitude, satellites, hdop, provider)
             VALUES ($1,$2,$3,$4,$5,$6,$7,$8,$9,$10)
             ON CONFLICT (vehicle_id, timestamp) DO NOTHING`,
            e.VehicleID, e.Timestamp, e.Lat, e.Lon, e.Speed,
            e.Heading, e.Altitude, e.Satellites, e.Hdop, e.Provider,
        )
    }
    conn.SendBatch(ctx, batch)  // одна round-trip, 300 строк
}
```

**Почему batch INSERT, а не individual INSERT и не COPY:**

| Метод | WAL flushes/sec | ON CONFLICT | Сложность |
|---|---|---|---|
| Individual INSERT | 1500 — PG деградирует | Да | Простой |
| **Batch INSERT (pgx.Batch)** | **5** | **Да, нативно** | **Простой** |
| COPY | 5 | **Нет** — нужна staging-таблица | Сложный |

COPY быстрее на 20-40% для миллионов строк (ETL, начальная загрузка). Но для 300 строк каждые 200 мс разница незаметна. Batch INSERT проще и поддерживает `ON CONFLICT` из коробки — не нужна temp-таблица, TRUNCATE, два шага.

**Параметры буфера:**
- **Размер: 300 событий.** 1500/sec ÷ 5 flush/sec = 300. Один batch каждые 200 мс.
- **Таймаут: 200 мс.** Если за 200 мс не набралось 300 — flush то что есть. Гарантирует что данные не застрянут в буфере.

**Порядок коммита:**
1. Batch INSERT в PG ✓
2. Commit Kafka offset ✓ — **только после** успешного INSERT

Если INSERT упал → offset не закоммичен → consumer перечитает те же события → `ON CONFLICT DO NOTHING`.

**COPY как план B:** если нагрузка вырастет в 10x (рост автопарка до 150K ТС или переход на 1 Hz для критичных грузов) и batch INSERT станет bottleneck — переходим на COPY через staging-таблицу. На текущих 1.5K/sec batch INSERT с запасом.

### Партиционирование

```sql
CREATE TABLE position_history (
    vehicle_id  UUID          NOT NULL,
    timestamp   BIGINT        NOT NULL,  -- unix ms
    lat         DOUBLE PRECISION NOT NULL,
    lon         DOUBLE PRECISION NOT NULL,
    speed       DOUBLE PRECISION,
    heading     DOUBLE PRECISION,
    altitude    DOUBLE PRECISION,
    satellites  INTEGER,
    hdop        DOUBLE PRECISION,
    provider    VARCHAR(20),
    PRIMARY KEY (vehicle_id, timestamp)
) PARTITION BY RANGE (timestamp);
```

**Ежемесячные партиции:**
```sql
-- Автоматическое создание через pg_partman:
SELECT partman.create_parent(
    'public.position_history',
    'timestamp',
    'native',
    'monthly'
);
```

**Как PG оптимизирует запросы с партиционированием:**
```sql
-- Запрос: маршрут truck-42 за 15 января 2026
SELECT lat, lon, speed, heading, timestamp
FROM position_history
WHERE vehicle_id = 'truck-42-uuid'
  AND timestamp BETWEEN 1736899200000 AND 1736985600000
ORDER BY timestamp;

-- PG query planner видит: timestamp попадает в партицию position_history_2026_01
-- Сканирует ТОЛЬКО эту партицию (partition pruning)
-- Использует B-tree индекс (vehicle_id, timestamp)
-- Результат: ~8 640 строк (1 event / 10 сек × 86 400 сек/день) за <20 мс
```

**Без партиционирования:** PG сканировал бы индекс по ВСЕЙ таблице (сотни миллионов строк). С партиционированием — только по одной партиции (~4M строк). Разница на порядок и больше.

### Репликация

**Async replica, НЕ sync.**

| | Tracking Service (GPS) | Order Service (заказы) |
|---|---|---|
| Данные | Координаты ТС | Заказы, деньги |
| Потеря при failover | 1-2 сек GPS — ок | Потеря заказа — недопустимо |
| Репликация | **Async** | **Sync** |
| Почему | GPS приходит каждую секунду, потерянное восстановится | Заказ создаётся один раз, потеря = бизнес-проблема |

**Sync replica для GPS означала бы:**
- Каждый COPY batch ждёт подтверждения от реплики
- Latency записи вырастает на ~1-5 мс (сетевой round-trip)
- При проблемах с репликой — запись блокируется
- Ради данных которые через секунду перезапишутся — бессмысленно

**Patroni** для автоматического failover:
- Master упал → Patroni промоутит async реплику за 10-30 сек
- Потеря: данные с момента последней репликации (~1-2 сек)
- Tracking Service получит ошибку записи → retry → новый master

**Read-реплика:**
- Диспетчерские read-запросы ("маршрут за вчера") идут на async реплику
- Master разгружен — только COPY batch writes
- Задержка реплики: ~100-500 мс (для GPS-истории некритично)

---

## Redis — текущие позиции (Hot storage)

### Что хранится

```
Ключ: vehicle:{vehicle_id}
Тип: Hash

vehicle:truck-42 = {
    lat:        "55.751244"
    lon:        "37.618423"
    speed:      "82.5"
    heading:    "45.0"
    altitude:   "156.0"
    satellites: "12"
    hdop:       "0.8"
    provider:   "wialon"
    timestamp:  "1704067200000"
}
```

### Почему Hash, а не String (JSON)

|                  | Hash                                                 | String (JSON)                         |
| ---------------- | ---------------------------------------------------- | ------------------------------------- |
| Частичное чтение | `HGET vehicle:truck-42 lat lon` — только нужные поля | Читаешь весь JSON, парсишь на клиенте |
| Частичная запись | `HSET vehicle:truck-42 lat 55.7 lon 37.6 ...`        | Перезапись всего JSON                 |
| Память           | Redis оптимизирует маленькие хэши (ziplist encoding) | Строка как есть                       |

При 1.5K полных перезаписей/sec разница небольшая. Но для read (диспетчер хочет только lat/lon) — Hash экономит.

### Нагрузка на Redis

```
Write: 1 500 HSET/sec          (обновление позиций)
Read:  ~50 HGETALL/sec          (одиночные запросы)
       ~250 pipeline HGETALL/sec (карта диспетчера, poll каждые 2 сек)
Total: ~1 800 ops/sec

Redis benchmark: один инстанс держит 100K+ ops/sec
→ 1.8K — 1.8% от capacity. Огромный запас.
```

### Память

```
15 000 ключей × ~200 байт (hash с ziplist encoding) = 3 МБ
+ Redis overhead (~50% для metadata) = ~5 МБ

5 МБ. Redis даже не заметит.
```

### TTL как "dead man switch"

**TTL: 5 минут.** Каждый HSET обновляет TTL. Если ТС перестаёт слать данные (выключено зажигание, трекер офлайн, провайдер лёг):
- 5 минут без обновлений → ключ исчезает
- Диспетчер запрашивает позицию → `nil` → на карте иконка "офлайн"

**Почему 5 минут, не 1:**
- Трекер может быть в тоннеле или зоне без связи (1-2 мин)
- Провайдер может тормозить (задержка доставки 30-60 сек)
- 5 минут покрывает все нормальные сценарии

### Redis Sentinel

```
Master (1) — все write + read
Replica (1) — standby для failover
Sentinels (3) — мониторинг + автоматический failover

НЕ Redis Cluster — 5 МБ данных, один инстанс справляется.
Cluster нужен для шардирования больших объёмов (100K+ ключей по ГБ).
```

**Master упал:**
- Sentinel детектит за 5-10 сек
- Промоутит реплику → новый master
- Tracking Service переподключается (клиент Redis с Sentinel-aware)
- Потеря: последние позиции (обновятся через 1 сек с новыми GPS-событиями)

---

## REST API для чтения позиций

### Контракт

| Endpoint | Источник данных | Кто использует |
|---|---|---|
| `GET /tracking/vehicles/{id}/current` | Redis `HGETALL vehicle:{id}` | Карточка ТС, клиентский ЛК, звонок от грузоотправителя |
| `GET /tracking/positions?vehicles=id1,id2,...` | Redis pipeline `HGETALL` батчем | Карта диспетчера (poll каждые 2 сек) |
| `GET /tracking/vehicles/{id}/history?from=&to=` | PG `position_history` с partition pruning | История маршрута, отчёты, расследования |

### Почему polling, а не WebSocket

Рассматривали WebSocket для real-time обновлений карты, но выбрали polling:

- **Stateless сервис.** WebSocket делает Tracking Service stateful: соединения привязаны к инстансу, HPA scale-in убивает 500 подключений одновременно, Kafka ребаланс не совпадает с распределением WS. Требует sticky sessions через NGINX, preStop hooks для graceful close, reconnect backoff на клиенте — сложный lifecycle. С polling Tracking Service остаётся полностью stateless, HPA работает прозрачно, под можно убивать без специальных процедур.
- **Low cost.** 500 диспетчеров × 1 poll каждые 2 сек = 250 RPS к Redis. Capacity Redis одного инстанса ~100K ops/sec → утилизация 0.25%. Никаких оптимизаций не требуется. Пикам не страшно — burst 5× → 1250 RPS = 1.25% capacity.
- **Latency некритична.** Задержка 0-2 сек на обновление точки на карте не заметна человеческому глазу — глаз не отличает движение с частотой 0.5 Гц от 1 Гц. WebSocket оправдан для latency < 100 мс (торговые системы, игры, чат) — для диспетчерского мониторинга оверкилл.
- **Операционная простота.** Один REST endpoint с чтением из Redis. Никаких процессов для close frame, нет проблемы "клиент отключился молча и висит на сервере час", нет специальных метрик `active_ws_connections`, нет отладки WS-специфики.
- **Throttling на клиенте.** Фронтенд сам решает когда поллить: активная карта — каждые 2 сек, свёрнутая вкладка — пауза (через `document.visibilityState === 'hidden'`). Экономит батарею мобильных клиентов и трафик.

### Нагрузка

```
500 диспетчеров × 1 poll / 2 сек = 250 RPS к /tracking/positions
Каждый poll: pipeline HGETALL по ~50 vehicle_id (среднее число ТС на экране карты)
Redis pipeline: 1 round-trip, ~2-5 мс на весь batch
CPU Tracking Service: <1% одного пода

+ ~50 RPS на /tracking/vehicles/{id}/current (одиночные запросы из карточек)
+ ~50 RPS на /tracking/vehicles/{id}/history (отчёты, расследования)

Итого: ~350 RPS read на весь сервис. Один под с запасом.
```

При росте до 5000 диспетчеров → 2500 RPS → всё ещё 2.5% Redis capacity. Потолок на read пока теоретический.

### Когда имело бы смысл вернуть WebSocket

Если потребуется sub-секундная реактивность на критичные события (SOS от водителя, выход за геозону в реальном времени) — отдельный канал push через Notification Service (который уже использует push-протоколы). Для обычного отображения точек на карте — polling с запасом.

---

## Масштабирование Tracking Service

### Consumer group + HPA

```yaml
Tracking Service:
  min replicas: 2
  max replicas: 4
  metric: kafka_consumer_lag (telematics.gps) > 200 → scale up
  secondary: CPU > 70% → scale up
```

**Потолок consumer-ов:** 8 (= партиций `telematics.gps`). Аналогично Telematics Service. Больше инстансов чем партиций → лишние idle.

### Bottleneck analysis

| Компонент | Текущая нагрузка | Capacity | Запас | Что делать при ×10 |
|---|---|---|---|---|
| **Redis** | 1.8K ops/sec | 100K+ ops/sec | ×55 | Маловероятно упереться — 5 МБ данных, даже Cluster не нужен |
| **PG write** | 5 batch/sec (300 rows) | ~50 batch/sec | ×10 | COPY через staging-таблицу; шардирование PG по `vehicle_id` (чётные/нечётные) |
| **PG storage** | 660 ГБ/month warm | ~2 ТБ manageable | ×3 | Уменьшить warm window (30 → 14 → 7 дней), потом шардирование |
| **Kafka consumer** | 1.5K/sec | ~50K/sec (8 партиций) | ×30 | Увеличить партиции (но ломает порядок по `vehicle_id`) |

**Главный bottleneck при росте: PG storage.** Решения по нарастанию:
1. Уменьшить warm window (30 → 14 → 7 дней) — сразу в 4 раза меньше warm
2. Перейти на COPY через staging при росте write нагрузки
3. Шардирование PG по `vehicle_id` (два кластера, чётные/нечётные)
4. Заменить PG warm на ClickHouse (если point query latency допустима)

---

## Сценарии отказа

### Инстанс Tracking Consumer упал

Аналогично Telematics Service:
```
t=0     Инстанс упал
t=0-6   Kafka ждёт heartbeat timeout
t=6-10  Ребаланс, партиции переезжают
t=10+   Другие инстансы подхватывают, перечитывают незакоммиченные события
        PG: ON CONFLICT DO NOTHING (дубли безопасны)
        Redis: SET перезапишет тем же значением
```

**Потеря in-flight batch:** инстанс копил 300 событий в buffer, упал до batch INSERT. Kafka offset не закоммичен → consumer перечитает те же 300 → batch INSERT повторно → `ON CONFLICT DO NOTHING`.

### PG master упал

```
t=0      Master недоступен
t=0-10   batch INSERT фейлятся, Tracking Service НЕ коммитит Kafka offset.
         In-flight события держатся в Kafka — lag растёт.
t=10-30  Patroni промоутит async реплику → новый master
t=30+    Tracking Service переподключается к PG, дочитывает Kafka
         с последнего committed offset → перечитанные события проходят
         через ON CONFLICT DO NOTHING, дубли игнорируются.

Потеря: 1-2 сек данных от async replication lag Patroni.
Это ~150-300 GPS-точек (1500/sec × 1-2 сек). Через 10 секунд придут свежие —
диспетчер не заметит.
```

**Почему не ring buffer в памяти Tracking Service:**
- Ring buffer = stateful: при OOM инстанса / рестарте пода буфер теряется.
- При длительном failover (5+ минут split-brain сценарий) буфер разрастается до сотен МБ → OOM.
- Kafka сам держит буфер через retention — надёжнее и бесплатно. Не коммитим offset → события никуда не денутся.
- Это та же философия что в Telematics Service: stateless, полагаемся на Kafka как буфер.

### Redis master упал

```
t=0      Master недоступен
t=0-10   HSET фейлятся, позиции не обновляются
         GET-запросы → nil (реплика может быть stale)
t=5-15   Sentinel промоутит реплику
t=15+    Позиции обновляются (GPS приходит каждую секунду)

Потеря: текущие позиции устаревают на 5-15 сек
На карте: точки "зависают" на 15 сек → потом обновляются
Критичность: низкая (диспетчер увидит короткий "фриз")
```

### Kafka (telematics.gps) недоступен

```
Один брокер упал:
  Автоматический failover лидера (секунды). Consumer переподключается.

Все брокеры упали:
  Consumer не может читать → Redis устаревает → PG не пополняется
  Данные не теряются — Telematics Service тоже не может писать,
  его буфер копится (или raw-адаптеры копят)

  После восстановления: consumer начинает с последнего offset → догоняет
```

### S3 (MinIO) недоступен при архивации

```
Cron пытается архивировать partition_2026_01:
  → S3 недоступен → retry через 1 час
  → Партиция остаётся в PG (занимает место, но не ломает ничего)
  → Алерт если не удалось за 24 часа
  → Критичность: низкая (PG просто хранит лишнюю партицию)
```

### Сводная таблица

| Сценарий | Потеря данных | Потеря доступности | Время восстановления |
|---|---|---|---|
| 1 инстанс Tracking упал | Нет (Kafka перечитает) | Нет (другие инстансы) | 6-10 сек (ребаланс) |
| PG master упал | 1-2 сек GPS (async) | Write: 10-30 сек. Read: → реплика | 10-30 сек (Patroni) |
| Redis master упал | Нет (обновится через 10 сек) | Read: stale 5-15 сек | 5-15 сек (Sentinel) |
| Kafka брокер упал | Нет (replication) | Нет (failover лидера) | Секунды |
| S3 недоступен | Нет | Нет (архивация отложена) | Retry через 1 час |

---

## Graceful shutdown

K8s при остановке пода: `SIGTERM` → grace period (`terminationGracePeriodSeconds: 30`) → `SIGKILL`. Tracking Service на shutdown:

1. **preStop hook** → HTTP POST `/admin/shutdown` → старт graceful sequence.
2. **Pause Kafka consumer** — прекращаем poll новых сообщений из `telematics.gps`.
3. **Flush batch INSERT буфера** — текущие 0-300 накопленных событий сразу летят в PG через `pgBatchInsert()`, ждём успешного ACK.
4. **Commit Kafka offset** — после успешного flush коммитим offset последних обработанных событий.
5. **Close Redis connection pool** — текущие in-flight HSET завершаются, новые не принимаются.
6. **REST API returns 503** — `/tracking/positions` и другие endpoints отдают 503 с `Retry-After: 5` через middleware. NGINX Ingress readinessProbe получает 503 → убирает под из пула, новые запросы идут на живые инстансы.
7. **Close consumer group membership** — Kafka координатор узнаёт о уходе и ребалансит сразу.
8. **Exit 0.**

Если не успели за 30 сек → SIGKILL → ребаланс по heartbeat timeout (6 сек), in-flight batch INSERT события перечитываются при восстановлении consumer и проходят через `ON CONFLICT DO NOTHING`. Максимум ~300 событий (один батч) могут быть обработаны повторно — это штатный at-least-once сценарий, downstream идемпотентен.

---

## Идемпотентность

Consumer at-least-once → может получить дубль. Все операции Tracking Service идемпотентны:

| Операция       | При дубле                                        | Почему безопасно              |
| -------------- | ------------------------------------------------ | ----------------------------- |
| Redis HSET     | Перезапись тем же значением                      | Noop — те же lat/lon/speed    |
| PG COPY        | `ON CONFLICT (vehicle_id, timestamp) DO NOTHING` | Строка уже есть, skip         |

---

## Ключевые решения

| Решение | Почему | Trade-off |
|---|---|---|
| PG, не ClickHouse | Point queries, команда знает PG, operational simplicity | Больше storage (660 ГБ warm vs ~60 ГБ в ClickHouse) |
| Hot/Warm/Cold (Redis/PG/S3) | 24 ТБ в одном PG операционно тяжело (VACUUM, бэкапы, index bloat) | Сложность архивации, cold данные недоступны мгновенно |
| Batch INSERT (pgx.Batch), не individual | 1.5K INSERT/sec через individual = 1500 WAL flush/sec, PG деградирует. Batch 300 строк / 200 мс = 5 flush/sec | Задержка до 200 мс (буфер) |
| Async replica, не sync | GPS — не деньги, 1-2 сек потери ок | При failover теряем ~1.5-3K точек (через 10 сек придут свежие) |
| Партиционирование по месяцам | Partition pruning, удобная архивация | Больше партиций = сложнее DDL |
| Redis TTL 5 мин | Dead man switch для "офлайн" статуса | ТС в тоннеле на 6 мин → ложный "офлайн" |
| REST polling вместо WebSocket | Stateless сервис, операционная простота, 0-2 сек latency незаметна глазу, polling 250 RPS = 0.25% Redis capacity | Нет true real-time push. Если понадобится (SOS, геозоны) — отдельный канал через Notification Service |

# Моя часть — Tracking Service

## Контекст

Tracking Service — потребитель `telematics.gps` из телематического pipeline. Получает нормализованные GPS-события от Telematics Service и делает три вещи:

1. **Write** — сохранить текущую позицию (Redis) и историю маршрута (PostgreSQL)
2. **Read** — отдать текущую позицию или историю по запросу диспетчера
3. **Push** — протолкнуть обновление позиции в WebSocket подключённым клиентам

```
telematics.gps (Kafka, ~15 000 events/sec)
       │
       ▼
Tracking Service (consumer group, 3-5 инстансов)
       │
       ├──→ Redis SET vehicle:{id}           (текущая позиция, hot)
       │
       ├──→ Buffer → PG COPY batch insert    (история маршрута, warm)
       │
       └──→ WebSocket fan-out                (диспетчерам, throttled)
```

---

## Нагрузка

### Входные данные

| Параметр | Значение |
|---|---|
| Источник | Kafka топик `telematics.gps` |
| RPS на входе | ~15 000 events/sec |
| Активных ТС | 15 000 |
| Формат | Protobuf (`RawTelematicsEvent`, type = GPS) |
| Retention истории | 3 года (требование заказчика, compliance) |

### Write path

| Операция | RPS | Куда |
|---|---|---|
| Обновление текущей позиции | 15 000 SET/sec | Redis |
| Запись в историю | 15 000 rows/sec | PostgreSQL (через COPY batch) |
| WebSocket push | ~500 fan-out/sec | Подключённые диспетчеры |

### Read path

| Операция | RPS | Откуда | Кто |
|---|---|---|---|
| "Где truck-42 сейчас?" | ~100 req/sec | Redis GET → O(1) | Диспетчеры, клиентский ЛК |
| "Все ТС на карте" | ~10 req/sec | Redis MGET / pipeline | Диспетчерская карта |
| "Маршрут truck-42 за вчера" | ~50 req/sec | PG → index scan | Диспетчеры, отчёты |

**Соотношение write/read: 15 000 / 160 ≈ 100:1.** Крайне write-heavy сервис.

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
Per second:  15 000 × 170 B = 2.55 MB/sec
Per day:     2.55 × 86 400  = 220 GB/day
Per month:   220 × 30       = 6.6 TB/month
Per year:    6.6 × 12       = 79 TB/year
Per 3 years:                = 237 TB (полный retention)
```

**237 TB за 3 года — это МНОГО для PostgreSQL.** Отсюда стратегия hot/warm/cold storage.

---

## Hot / Warm / Cold — трёхуровневое хранение

### Зачем три уровня

237 TB в одном PG — нереально. Диски, бэкапы, vacuum, индексы — всё ломается. Но 95% запросов диспетчеров — за последние 24-48 часов. Старые данные нужны только для compliance.

### Три уровня

```
┌──────────────────────────────────────────────────────────────┐
│ HOT: Redis                                                   │
│ Текущая позиция каждого ТС                                   │
│ 15 000 ключей × 200 байт = 3 МБ                             │
│ Запросы: "Где truck-42 сейчас?" → GET → O(1), <1 мс         │
│ TTL: 5 мин (dead man switch — нет данных 5 мин = "офлайн")  │
└──────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│ WARM: PostgreSQL                                             │
│ История за последние 30 дней                                 │
│ ~6.6 TB, ежемесячные партиции                                │
│ Запросы: "Маршрут truck-42 за вчера" → index scan, <50 мс   │
│ Индекс: (vehicle_id, timestamp) → partition pruning          │
└──────────────────────────────────────────────────────────────┘
                         │
              архивация (cron, 1 раз/месяц)
                         ▼
┌──────────────────────────────────────────────────────────────┐
│ COLD: S3 (MinIO)                                             │
│ Архив: 30+ дней, compressed pg_dump                          │
│ ~230 TB за 3 года (с compression ~2-3x → ~80-115 TB)        │
│ Запросы: почти никогда. Compliance, legal, инциденты         │
│ Восстановление: pg_restore в temp таблицу → запрос → drop    │
└──────────────────────────────────────────────────────────────┘
```

### Почему 30 дней warm, не 90

| Вариант | Объём в PG | Проблемы |
|---|---|---|
| 7 дней | 1.5 TB | Мало — ежемесячные отчёты не покрываются |
| **30 дней** | **6.6 TB** | **Покрывает 95% запросов + ежемесячные отчёты** |
| 90 дней | 20 TB | PG тяжело: vacuum, бэкапы, pg_dump долгий |
| 365 дней | 79 TB | Нереально для PG |

30 дней — sweet spot: покрывает операционные нужды, PG справляется.

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
- Скачивание из S3: ~5-15 минут (6.6 TB / партиция, compressed ~2-3 TB)
- pg_restore: ~10-30 минут
- Итого: **~15-45 минут.** Для compliance/legal — допустимо.

**Автоматизация (если восстановления частые):**
API-эндпоинт: `POST /admin/archive/restore?month=2026-01` → фоновый job → Kafka event → Slack notification когда готово. Но у нас такого не было — восстановления были ~1 раз в квартал.

---

## Выбор БД — почему PostgreSQL, а не ClickHouse

### Контекст

15K write/sec и 237 TB за 3 года — числа, при которых ClickHouse напрашивается. Мы обсуждали оба варианта.

### Сравнение

| Критерий | PostgreSQL | ClickHouse |
|---|---|---|
| **Тип запросов Tracking** | Point query: "маршрут truck-42, 10:00-12:00 сегодня" | Scan query: "средняя скорость всех ТС за квартал" |
| **Write** | COPY batch 15K/sec — справляется | Native batch — тоже справляется |
| **Point query latency** | <50 мс (B-tree index на (vehicle_id, timestamp)) | ~200-500 мс (sparse index, full block scan) |
| **Compression** | 2-3x | **10-50x** (237 TB → 20-50 TB) |
| **Storage за 3 года** | ~80-115 TB (с archival в S3) | ~20-50 TB (всё в ClickHouse) |
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

**4. Storage решается hot/warm/cold.**

237 TB "в лоб" в PG — да, нереально. Но с archival: 6.6 TB warm + S3 cold. Это управляемо.

### Trade-off (честно для интервью)

"Мы осознанно выбрали больший объём хранения (PG + S3 vs ClickHouse), потому что это экономило время команды — zero learning curve. Если бы хранение стало дорогим или аналитические запросы по историческим данным стали частыми — ClickHouse как замена warm-слоя был следующим шагом. Но за время проекта до этого не дошло."

---

## PostgreSQL — детали

### Batch INSERT

**Проблема:** 15K individual INSERT/sec = 15K транзакций = 15K WAL flush-ей. PostgreSQL ляжет — WAL writer станет bottleneck, latency вырастет до секунд.

**Решение: буфер + batch INSERT.**

```go
// В каждом инстансе Tracking Service:
buffer := make([]PositionEvent, 0, 3000)

for event := range kafkaConsumer {
    buffer = append(buffer, event)

    if len(buffer) >= 3000 || timeSinceLastFlush > 200*time.Millisecond {
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
    conn.SendBatch(ctx, batch)  // одна round-trip, 3000 строк
}
```

**Почему batch INSERT, а не individual INSERT и не COPY:**

| Метод | WAL flushes/sec | ON CONFLICT | Сложность |
|---|---|---|---|
| Individual INSERT | 15 000 — PG умрёт | Да | Простой |
| **Batch INSERT (pgx.Batch)** | **5** | **Да, нативно** | **Простой** |
| COPY | 5 | **Нет** — нужна staging-таблица | Сложный |

COPY быстрее на 20-40% для миллионов строк (ETL, начальная загрузка). Но для 3000 строк каждые 200 мс разница ~1-2 мс — незаметна. Batch INSERT проще и поддерживает `ON CONFLICT` из коробки — не нужна temp-таблица, TRUNCATE, два шага.

**Параметры буфера:**
- **Размер: 3000 событий.** 15K/sec ÷ 5 flush/sec = 3000. Один batch каждые 200 мс.
- **Таймаут: 200 мс.** Если за 200 мс не набралось 3000 — flush то что есть. Гарантирует что данные не застрянут в буфере.

**Порядок коммита:**
1. Batch INSERT в PG ✓
2. Commit Kafka offset ✓ — **только после** успешного INSERT

Если INSERT упал → offset не закоммичен → consumer перечитает те же события → `ON CONFLICT DO NOTHING`.

**COPY как план B:** если нагрузка вырастет в 10x и batch INSERT станет bottleneck — переходим на COPY через staging-таблицу. Но при 15K/sec — batch INSERT с запасом.

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
-- Результат: ~86 400 строк (1 сек × 86 400 сек/день) за <50 мс
```

**Без партиционирования:** PG сканировал бы индекс по ВСЕЙ таблице (миллиарды строк). С партиционированием — только по одной партиции (~40M строк). Разница — на порядки.

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

| | Hash | String (JSON) |
|---|---|---|
| Частичное чтение | `HGET vehicle:truck-42 lat lon` — только нужные поля | Читаешь весь JSON, парсишь на клиенте |
| Частичная запись | `HSET vehicle:truck-42 lat 55.7 lon 37.6 ...` | Перезапись всего JSON |
| Память | Redis оптимизирует маленькие хэши (ziplist encoding) | Строка как есть |

При 15K полных перезаписей/sec разница небольшая. Но для read (диспетчер хочет только lat/lon) — Hash экономит.

### Нагрузка на Redis

```
Write: 15 000 HSET/sec (обновление позиций)
Read:  ~100 HGETALL/sec (диспетчеры) + ~10 MGET/sec (карта)
Total: ~15 110 ops/sec

Redis benchmark: один инстанс держит 100K+ ops/sec
→ 15K — 15% от capacity. Огромный запас.
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

## WebSocket — push обновлений

### Зачем

Диспетчер открыл карту с 200 ТС. Без WebSocket — поллинг: 200 запросов каждую секунду = 200 RPS × N диспетчеров. С WebSocket — сервер пушит обновления только когда позиция изменилась.

### Как работает

```
1. Диспетчер подключается: WS /tracking/subscribe?vehicles=truck-42,truck-43,...
2. Tracking Service регистрирует подписку: subscriber → [vehicle_ids]
3. При обновлении позиции truck-42:
   → Redis HSET (текущая позиция)
   → PG buffer (история)
   → Найти подписчиков truck-42 → push в их WS connections
```

### Throttling

**НЕ отправляем каждое из 15K events/sec в WebSocket.** GPS приходит каждую секунду на каждое ТС, а человеку достаточно обновления раз в 1-3 секунды. Карта не отрендерит быстрее.

```
Throttle: максимум 1 update/sec per vehicle per subscriber

15 000 events/sec → после throttle: ~5 000 WS messages/sec (при 500 подписчиках × 10 ТС в среднем)
```

### Масштабирование WebSocket

**Проблема:** WebSocket — stateful. Подключение привязано к конкретному инстансу Tracking Service. При ребалансе Kafka — партиции переезжают, но WS-подключения остаются.

**Решение:** разделить consumer-ов и WS-обработчики.

```
Tracking Consumer (×3-5, stateless):
  Kafka → Redis + PG + Kafka "tracking.position-updated"

Tracking WS Gateway (×2-3, stateful):
  Kafka "tracking.position-updated" → WebSocket push

Разделение:
  - Consumer масштабируется по consumer group (Kafka ребаланс)
  - WS Gateway масштабируется независимо (sticky sessions через NGINX)
  - WS Gateway тоже consumer — читает из tracking.position-updated
```

Если нагрузка на WS маленькая (~500 подписчиков) — можно совместить в одном сервисе. Разделять когда WS подключений > 5 000.

---

## Масштабирование Tracking Service

### Consumer group + HPA

```yaml
Tracking Consumer:
  min replicas: 3
  max replicas: 8
  metric: kafka_consumer_lag (telematics.gps) > 1000 → scale up
  secondary: CPU > 70% → scale up

Tracking WS Gateway (если отдельный):
  min replicas: 2
  max replicas: 4
  metric: active_ws_connections > 1000 per pod → scale up
```

**Потолок consumer-ов:** 12 (= партиций `telematics.gps`). Аналогично Telematics Service.

### Bottleneck analysis

| Компонент | Текущая нагрузка | Capacity | Запас | Что делать при ×10 |
|---|---|---|---|---|
| **Redis** | 15K ops/sec | 100K+ ops/sec | ×6 | Redis Cluster (но маловероятно — 5 МБ) |
| **PG write** | 5 COPY/sec | ~50 COPY/sec | ×10 | Шардирование PG по vehicle_id |
| **PG storage** | 6.6 TB/month warm | ~20 TB manageable | ×3 | Уменьшить warm window до 7 дней |
| **Kafka consumer** | 15K/sec | ~100K/sec (12 партиций) | ×6 | Добавить партиции |
| **WebSocket** | 500 connections | ~10K/pod | ×20 | Отдельный WS Gateway |

**Главный bottleneck при росте: PG storage.** Решения по нарастанию:
1. Уменьшить warm window (30 → 14 → 7 дней)
2. Шардирование PG по vehicle_id (два кластера, чётные/нечётные)
3. Заменить PG warm на ClickHouse (если point query latency допустима)

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

**Потеря буфера COPY:** инстанс копил 3000 событий в buffer, упал до COPY. Kafka offset не закоммичен → consumer перечитает → COPY повторно → `ON CONFLICT DO NOTHING`.

### PG master упал

```
t=0      Master недоступен
t=0-10   COPY batch-и фейлятся, Tracking Service буферизует (ring buffer в памяти)
t=10-30  Patroni промоутит async реплику → новый master
t=30+    Tracking Service переподключается, flush буфер

Потеря: 1-2 сек данных (async replication lag)
Это ~15 000-30 000 GPS-точек. Через секунду придут свежие — диспетчер не заметит.
```

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
| Redis master упал | Нет (обновится через 1 сек) | Read: stale 5-15 сек | 5-15 сек (Sentinel) |
| Kafka брокер упал | Нет (replication) | Нет (failover лидера) | Секунды |
| S3 недоступен | Нет | Нет (архивация отложена) | Retry через 1 час |

---

## Идемпотентность

Consumer at-least-once → может получить дубль. Все операции Tracking Service идемпотентны:

| Операция       | При дубле                                        | Почему безопасно              |
| -------------- | ------------------------------------------------ | ----------------------------- |
| Redis HSET     | Перезапись тем же значением                      | Noop — те же lat/lon/speed    |
| PG COPY        | `ON CONFLICT (vehicle_id, timestamp) DO NOTHING` | Строка уже есть, skip         |
| WebSocket push | Повторный push той же позиции                    | Клиент перерисует ту же точку |

---

## Ключевые решения

| Решение | Почему | Trade-off |
|---|---|---|
| PG, не ClickHouse | Point queries, команда знает PG, operational simplicity | Больше storage (6.6 TB warm vs ~0.6 TB в ClickHouse) |
| Hot/Warm/Cold (Redis/PG/S3) | 237 TB в одном PG нереально | Сложность архивации, cold данные недоступны мгновенно |
| COPY batch, не INSERT | 15K INSERT/sec = WAL bottleneck | Задержка до 200 мс (буфер) |
| Async replica, не sync | GPS — не деньги, 1-2 сек потери ок | При failover теряем ~30K точек |
| Партиционирование по месяцам | Partition pruning, удобная архивация | Больше партиций = сложнее DDL |
| Redis TTL 5 мин | Dead man switch для "офлайн" статуса | ТС в тоннеле на 6 мин → ложный "офлайн" |
| WebSocket throttle 1/sec | Человеку не нужно 15K fps | Задержка до 1 сек на карте |

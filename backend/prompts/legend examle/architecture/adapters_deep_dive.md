# Адаптеры — Deep Dive

Детальный разбор каждого из 5 адаптеров телематического pipeline. Общая архитектура pipeline описана в [[my_part_adapters]].

---

## Общий контракт

Каждый адаптер — отдельный stateless микросервис. Задача одна:

```
Формат провайдера → RawTelematicsEvent (protobuf) → Kafka telematics.raw
```

**Что есть у каждого адаптера:**
- Kafka producer (async, `acks=all`)
- Prometheus метрики
- Health check (`/healthz`, `/readyz`)
- Graceful shutdown

**Чего НЕТ ни у одного адаптера:**
- PostgreSQL — не нужен, нечего хранить
- Redis — не нужен, нет кеша
- Persistent state — при рестарте начинаем с чистого листа

### Внутренняя структура (одинаковая у всех)

```
Receiver          Parser            Mapper              KafkaProducer
(протокол         (формат           (структура          (protobuf →
провайдера)       провайдера →      провайдера →         telematics.raw)
                  Go struct)        RawTelematicsEvent)
```

- **Receiver** — уникален для каждого адаптера (WS-клиент / HTTP-сервер / TCP-сервер / REST-поллер / SOAP-клиент)
- **Parser** — уникален (JSON / binary / XML)
- **Mapper** — уникален (поля провайдера → поля protobuf)
- **KafkaProducer** — общий, один на все адаптеры (shared library)

### Маппинг provider_device_id → vehicle_id

Провайдер знает свой ID устройства (`tracker_123`). Мы знаем `vehicle_id` (UUID в нашей системе). Нужен маппинг.

```
При старте адаптера:
  → gRPC запрос к Vehicle Service: GetMappings(provider="navixy")
  → ответ: map[provider_device_id]vehicle_id (загружается в memory)

Обновления:
  → Kafka consumer: топик vehicle.mapping-updated
  → при добавлении/удалении ТС — маппинг обновляется в памяти

Если device_id не найден в маппинге:
  → событие дропается + метрика adapter_unknown_device_total{provider}
  → алерт если > 100/мин (массовая проблема — скорее всего обновили парк, а маппинг не обновился)
```

**Почему в памяти, а не запрос к Vehicle Service на каждое событие:**
~2K events/sec суммарно (1.5K GPS + 500 engine + 30 driver events) → ~2K gRPC calls/sec к Vehicle Service на каждое событие. Даже при таком потоке Vehicle Service станет узким местом и network round-trip добавит >1 мс latency к каждому событию. Маппинг меняется раз в день (добавили/убрали ТС из парка) — кеш в памяти с periodic refresh из Kafka event-ов обновления парка — разумный вариант.

### Graceful Shutdown (у всех адаптеров)

```
1. SIGTERM от K8s (preStop hook, 30 сек grace period)
2. Receiver: перестаёт принимать новые данные
   - WS/TCP: close connection
   - HTTP server: stop accepting
   - Pull: прекращаем polling
3. Drain: обрабатываем то что уже в буфере (channel)
4. Kafka producer: Flush() — дождаться подтверждения всех in-flight сообщений
5. os.Exit(0)

Timeout: 25 сек (оставляем 5 сек для K8s). Если не успели — K8s шлёт SIGKILL.
Потеря: то что было в буфере и не ушло в Kafka. Максимум — несколько сотен событий.
Последствия: pull-адаптеры заберут повторно, push-адаптеры — провайдер дошлёт при реконнекте.
```

---

## 1. Navixy Adapter (WebSocket, push)

### Как работает

```
Navixy Cloud
    │
    │  persistent WebSocket connection
    │  wss://api.navixy.com/v2/stream
    │
    ▼
Navixy Adapter (WS client)
    │
    │  JSON → protobuf
    │
    ▼
Kafka: telematics.raw
```

**Модель:** мы — клиент, Navixy — сервер. Адаптер открывает WS-соединение к API Navixy и получает поток событий. Navixy стримит данные со всех ТС, подключённых к их платформе, в реальном времени.

### Протокол

- **Транспорт:** WebSocket (RFC 6455) поверх TLS
- **Аутентификация:** API key в query string при handshake (`?hash=<api_key>`)
- **Формат сообщений:** JSON, массив событий в одном WS-фрейме
- **Heartbeat:** WS ping/pong, интервал 30 сек

### Формат входящих данных

```json
[
  {
    "tracker_id": "tracker_4821",
    "event_type": "position",
    "gps": {
      "lat": 55.751244,
      "lng": 37.618423,
      "speed": 82,
      "course": 45,
      "altitude": 156,
      "satellites": 12
    },
    "timestamp": "2026-01-15T10:30:00Z"
  },
  {
    "tracker_id": "tracker_4821",
    "event_type": "sensor",
    "sensors": {
      "fuel_level": 45.2,
      "engine_rpm": 2100,
      "ignition": true,
      "odometer": 142350.5
    },
    "timestamp": "2026-01-15T10:30:00Z"
  }
]
```

**Один WS-фрейм может содержать несколько событий от разных ТС.** Адаптер разбирает массив и генерит отдельный `RawTelematicsEvent` на каждое.

### Маппинг в protobuf

```
Navixy event_type    →  RawTelematicsEvent.EventType
─────────────────────────────────────────────────────
"position"           →  GPS
"sensor"             →  ENGINE
"alarm"              →  DRIVER_EVENT

Navixy поля          →  Protobuf поля
─────────────────────────────────────────────────────
gps.lat              →  lat
gps.lng              →  lon          (lng → lon, переименование)
gps.speed            →  speed        (км/ч, как есть)
gps.course           →  heading      (course → heading)
gps.altitude         →  altitude
gps.satellites       →  satellites
sensors.fuel_level   →  fuel_level   (%, как есть)
sensors.engine_rpm   →  engine_rpm
sensors.ignition     →  ignition
sensors.odometer     →  odometer
alarm.type           →  driver_event_code (маппинг кодов: "harsh_braking" → "harsh_brake")
```

### Edge Cases

#### WS disconnect (провайдер закрыл соединение / сеть пропала)

```
Причины: провайдер перезагружает API, сетевой сбой, TLS expired, maintenance window.

Поведение:
  t=0     WS read() вернул ошибку (io.EOF / websocket.CloseError)
  t=0     Логируем: adapter_ws_disconnect_total{provider="navixy"}
  t=1     Reconnect попытка 1 (backoff: 1 сек)
  t=3     Reconnect попытка 2 (backoff: 2 сек)
  t=7     Reconnect попытка 3 (backoff: 4 сек)
  t=15    Reconnect попытка 4 (backoff: 8 сек)
  t=30    Reconnect попытка 5 (backoff: 15 сек)
  t=45+   Попытки каждые 30 сек (max backoff)

Exponential backoff: 1s → 2s → 4s → 8s → 15s → 30s (cap)
Jitter: ±20% (чтобы при массовом disconnect не все адаптеры реконнектились одновременно)

Потеря данных за время disconnect:
  - Navixy буферизует на своей стороне (по контракту API)
  - После реконнекта: Navixy досылает buffered events (replay window ~60 сек)
  - Дубли (events отправлены до disconnect, но мы не подтвердили):
    downstream идемпотентен — Tracking через ON CONFLICT (vehicle_id, timestamp) DO NOTHING,
    Redis HSET overwrite, ClickHouse ReplacingMergeTree
  - Если disconnect > 60 сек: Navixy может потерять буфер → gap в данных
    → метрика adapter_gap_detected_total, алерт
```

#### Провайдер лёг полностью (HTTP 503, WS handshake failed)

```
Все попытки reconnect фейлятся.

  t=30s   Circuit breaker: 5 ошибок подряд → state = OPEN
          Перестаём пытаться подключиться (бессмысленно спамить мёртвый сервер)
          Метрика: adapter_circuit_breaker_state{provider="navixy"} = 1 (open)

  t=30s+  /readyz возвращает 503 → K8s НЕ шлёт трафик (но и не убивает — это liveness, не readiness)
          Алерт: "Navixy adapter circuit breaker OPEN > 1 мин"

  t=60s   Half-open: пробуем одно соединение (probe)
          Если успешно → CLOSED, нормальная работа
          Если нет → OPEN ещё на 30 сек

Последствия:
  - ТС подключённые через Navixy — невидимы на карте (Redis TTL 5 мин → "офлайн")
  - Данные теряются на стороне Navixy (их буфер ограничен)
  - Остальные 4 адаптера работают нормально — изоляция отказов
```

#### Backpressure (Kafka медленный / недоступен)

```
Navixy стримит ~400 events/sec (≈ доля Navixy от общих 2K events/sec).
Адаптер парсит и кладёт в channel перед Kafka producer.

  channel := make(chan *RawTelematicsEvent, 5000)

Нормальный режим:
  Navixy → channel (запись) → Kafka producer горутина (чтение) → Kafka
  Channel occupancy: ~20 (Kafka забирает быстрее чем Navixy шлёт)

Kafka тормозит:
  Channel заполняется: 20 → 500 → 2000 → 5000
  Channel полный (5000):
    → select с default: дроп самого старого события из channel
    → метрика: adapter_events_dropped_total{provider="navixy", reason="backpressure"}
    → лучше потерять старый GPS чем заблокировать WS reader (и потерять соединение)

Kafka полностью лёг:
  → channel заполняется за ~12 сек при 400 events/sec
  → дропаем ~400 events/sec
  → метрика → алерт "adapter Navixy dropping > 100 events/sec"
  → WS соединение живо — после восстановления Kafka поток возобновляется мгновенно
```

#### WS ping/pong timeout

```
WebSocket RFC: ping/pong для проверки что соединение живо.

Адаптер отправляет WS ping каждые 30 сек.
Ожидаем pong в течение 10 сек.

Если pong не пришёл:
  → соединение считается мёртвым (может TCP-соединение есть, но сервер завис)
  → close connection → reconnect с backoff
  → без этого: silent connection — TCP-соединение живо, но данные не приходят,
    и мы бы не узнали об этом минутами
```

#### Невалидный JSON в WS-фрейме

```
Причина: баг на стороне Navixy, corruption при передаче.

  → json.Unmarshal() вернул ошибку
  → дропаем ЭТОТ фрейм (не весь stream)
  → метрика: adapter_parse_errors_total{provider="navixy"}
  → логируем первые 500 байт фрейма для debug
  → WS-соединение НЕ рвём — следующий фрейм может быть валидным
  → алерт если > 10 parse errors/мин (систематическая проблема)
```

#### Navixy прислал событие с неизвестным tracker_id

```
tracker_id нет в маппинге provider_device_id → vehicle_id.

Причины:
  - Новый трекер установлен, но маппинг не обновлён
  - Navixy шлёт данные от чужого клиента (баг их API)

  → дроп события + метрика adapter_unknown_device_total{provider="navixy"}
  → при > 100 unknown/мин → алерт
  → обновление маппинга: Kafka event vehicle.mapping-updated → hot reload без рестарта
```

#### Navixy поменял формат API (breaking change)

```
Было: gps.lng (longitude)
Стало: gps.longitude

Парсер использует json struct tags с fallback:

  type NavixyGPS struct {
      Lat       float64 `json:"lat"`
      Lon       float64 `json:"lng"`       // основное поле
      LonNew    float64 `json:"longitude"` // fallback для новой версии
  }

  // После парсинга:
  lon := gps.Lon
  if lon == 0 && gps.LonNew != 0 {
      lon = gps.LonNew
  }

Более серьёзные изменения (структура полностью другая):
  → парсер вернёт ошибку → дроп → алерт → ручной апдейт адаптера
  → но это видно на дашборде за минуты, не за часы
```

---

## 2. СКАУТ Adapter (REST JSON, pull)

### Как работает

```
СКАУТ API (REST)
    ▲
    │  HTTP GET /api/events?from=<timestamp>&limit=1000
    │  каждые 1 сек
    │
СКАУТ Adapter (HTTP client, poller)
    │
    │  JSON → protobuf
    │
    ▼
Kafka: telematics.raw
```

**Модель:** мы — клиент, СКАУТ — сервер. Адаптер поллит REST API провайдера раз в секунду, забирает батч новых событий. Мы контролируем частоту — провайдер не может нас заспамить.

### Протокол

- **Транспорт:** HTTPS, REST JSON
- **Аутентификация:** API key в `Authorization` header
- **Формат:** JSON массив событий
- **Пагинация:** query parameter `from=<unix_ms>`, ответ отсортирован по timestamp

### Pull Logic — как работает поллинг

```go
var lastTimestamp int64 // в памяти, persistent state НЕТ

for {
    events, err := scoutClient.GetEvents(ctx, lastTimestamp, 1000)
    if err != nil {
        // retry (см. edge cases ниже)
        continue
    }

    for _, event := range events {
        proto := mapToProtobuf(event)
        kafkaProducer.Send("telematics.raw", proto)
    }

    if len(events) > 0 {
        lastTimestamp = events[len(events)-1].Timestamp
    }

    time.Sleep(1 * time.Second) // poll interval
}
```

**`lastTimestamp` хранится в памяти.** Не в Redis, не в файле. При рестарте — теряется. Это by design (см. edge case "рестарт").

### Формат входящих данных

```json
{
  "events": [
    {
      "device_id": "scout_7792",
      "type": "gps",
      "data": {
        "latitude": 55.751244,
        "longitude": 37.618423,
        "speed_kmh": 82,
        "direction": 45,
        "alt": 156,
        "sat_count": 12,
        "hdop": 0.8
      },
      "ts": 1704067200000
    }
  ],
  "has_more": false,
  "next_from": 1704067201000
}
```

### Edge Cases

#### HTTP 429 (Rate Limit)

```
Провайдер ограничивает частоту запросов.

  → Читаем header Retry-After (если есть)
  → Backoff: если Retry-After не указан → ждём 5 сек → пробуем снова
  → Если 429 повторяется → увеличиваем интервал поллинга: 1s → 2s → 5s → 10s
  → Метрика: adapter_rate_limited_total{provider="scout"}
  → После успешного запроса → интервал возвращается к 1 сек

Почему не падаем:
  Данные копятся на стороне СКАУТ. Когда rate limit снимется — заберём всё за один батч.
  Задержка: секунды-минуты. Для GPS — допустимо.
```

#### HTTP 500 / 502 / 503

```
Провайдер лежит или деградирует.

  → Retry с exponential backoff: 1s → 2s → 4s → 8s → 15s → 30s (cap)
  → lastTimestamp НЕ меняем — при успешном retry заберём все пропущенные события
  → Circuit breaker: 5 ошибок за 30 сек → OPEN → probe каждые 30 сек
  → Метрика: adapter_http_errors_total{provider="scout", status="500"}
```

#### HTTP timeout (сеть / провайдер тормозит)

```
Timeout на HTTP-клиенте: 5 сек.

  → context.WithTimeout(ctx, 5*time.Second)
  → Timeout → та же логика что 500 (retry, не менять lastTimestamp)
  → Метрика: adapter_http_timeout_total{provider="scout"}

Почему 5 сек:
  - Нормальный ответ: 50-200 мс
  - 5 сек = что-то явно не так
  - Больше 5 сек → рискуем копить отставание (poll каждые 1 сек, а ответ идёт 10 сек)
```

#### Пустой ответ (нет новых событий)

```
{
  "events": [],
  "has_more": false
}

→ Нормальная ситуация: ночь, ТС стоят.
→ lastTimestamp не меняем.
→ Следующий poll через 1 сек как обычно.
→ Не логируем, не считаем ошибкой.
```

#### Пагинация (батч > 1000 событий)

```
Провайдер отдаёт максимум 1000 событий за запрос.
Если has_more=true — есть ещё.

  while has_more:
      events = GET /events?from=lastTs&limit=1000
      process(events)
      lastTs = events[-1].timestamp
      has_more = response.has_more

Когда это бывает:
  - Адаптер был выключен 5 минут → накопилось 5 мин × ~400 events/sec (доля СКАУТ) = ~120K событий
  - Забираем порциями по 1000, без sleep между страницами (catch-up mode)
  - Когда has_more=false → возвращаемся к обычному polling раз в 1 сек
```

#### Адаптер рестартнулся (потеря lastTimestamp)

```
lastTimestamp хранится в памяти → при рестарте = 0.

Стратегия: начинаем с from = now() - 30 сек.

Почему 30 сек overlap:
  - Покрывает время рестарта (K8s: kill → pull image → start = 5-15 сек)
  - Дубли за 30 сек: ~400 events/sec × 30 = ~12K событий. Downstream обрабатывает штатно
    (Tracking через ON CONFLICT, Redis HSET overwrite) — ~2 МБ лишних WAL на попытки INSERT
  - Больше 30 сек: лишняя нагрузка на провайдера и Kafka, а выигрыш минимальный

Почему НЕ храним lastTimestamp в Redis / PG:
  - Адаптер stateless by design. Добавлять зависимость ради одного int64 — overkill
  - Overlap + идемпотентность downstream = тот же результат, без внешней зависимости
  - Trade-off: ~12K дублей при рестарте ≈ ~2 МБ лишних ON CONFLICT попыток в PG.
    Дешевле, чем persistent state в адаптере
```

#### Clock skew (часы провайдера расходятся с нашими)

```
from=<timestamp> — это timestamp ИЗ ДАННЫХ провайдера, не наших часов.
Но что если часы СКАУТ убежали на 5 минут вперёд?

Проблема:
  Мы запрашиваем from=10:30:00 (наш lastTimestamp).
  У СКАУТ часы показывают 10:35:00 → события записаны с timestamps из будущего.
  Ответ: "нет событий после 10:30:00" → пустой.
  Реальные данные с ts 10:35:XX — мы их не заберём до 10:35:00 по нашим часам.

Результат: задержка доставки = clock skew (5 минут).
Не потеря — данные придут, но с задержкой.

Обнаружение:
  Метрика: received_at - timestamp. Если разница > 30 сек → алерт.
  Но тут разница будет ОТРИЦАТЕЛЬНОЙ (данные из "будущего") — тоже алертим.
```

#### Провайдер поменял формат (добавил/убрал поле)

```
JSON парсер на Go: неизвестные поля игнорируются по умолчанию.

Добавили поле:
  → json.Unmarshal пропустит неизвестное поле → ничего не сломается
  → если поле важное — добавим в парсер при следующем деплое

Убрали поле:
  → поле в Go struct будет zero value (0, "", false)
  → маппер проверяет: если lat == 0 && lon == 0 → дроп события (невалидные координаты)
  → если убрали некритичное поле (altitude) → событие пройдёт с altitude=0

Изменили тип поля (было число, стало строка):
  → json.Unmarshal вернёт ошибку
  → дроп события + метрика + алерт
```

---

## 3. Omnicomm Adapter (Webhook, push)

### Как работает

```
Omnicomm Server
    │
    │  HTTP POST на наш endpoint
    │  при каждом событии от трекера
    │
    ▼
Omnicomm Adapter (HTTP server)
    │
    │  JSON → protobuf
    │
    ▼
Kafka: telematics.raw
```

**Модель:** мы — сервер, Omnicomm — клиент. Провайдер POST-ит webhook на наш HTTP endpoint при каждом событии. Мы принимаем, парсим, отправляем в Kafka.

### Протокол

- **Транспорт:** HTTPS (наш TLS-сертификат, Let's Encrypt / cert-manager в K8s)
- **Аутентификация:** HMAC-SHA256 signature в header `X-Omnicomm-Signature`
- **Формат:** JSON body
- **Подтверждение:** HTTP 200 = принято. Любой другой код = Omnicomm ретраит

### HMAC-аутентификация

```go
func verifyHMAC(body []byte, signature string, secret []byte) bool {
    mac := hmac.New(sha256.New, secret)
    mac.Write(body)
    expected := hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expected), []byte(signature))
}

// В handler:
body, _ := io.ReadAll(r.Body)
sig := r.Header.Get("X-Omnicomm-Signature")
if !verifyHMAC(body, sig, sharedSecret) {
    w.WriteHeader(401)
    metrics.AuthFailures.Inc()
    return
}
```

**Shared secret** — в K8s Secret, монтируется как env variable. Ротация: обновляем Secret → rolling restart подов.

### Ключевое решение: когда отвечать 200

```
Вариант A (опасный):
  1. Получили webhook
  2. Ответили 200
  3. Отправили в Kafka
  Проблема: если между шагами 2 и 3 Kafka недоступен → мы ответили 200,
  но событие потеряно. Провайдер не ретраит (он получил 200).

Вариант B (наш):
  1. Получили webhook
  2. Отправили в Kafka (sync, ждём ack)
  3. Kafka подтвердил → ответили 200
  Если Kafka недоступен → ответили 500 → провайдер ретраит

Цена: latency ответа вырастает на ~5-10 мс (round-trip до Kafka).
Для webhook это нормально — провайдер не ожидает ответа за 1 мс.
```

### Edge Cases

#### HMAC не совпадает

```
Причины:
  1. Провайдер ротировал ключ, мы не обновили
  2. Replay attack / injection attempt
  3. Баг на стороне провайдера (неправильно считают HMAC)

  → 401 Unauthorized
  → метрика: adapter_auth_failures_total{provider="omnicomm"}
  → логируем: IP источника, первые 200 байт body (для debug), timestamp
  → алерт: > 10 auth failures/мин → возможна ротация ключа или атака

НЕ принимаем событие. Никаких fallback-ов. Безопасность > доступность данных.
```

#### Kafka недоступен → что отвечаем провайдеру

```
Провайдер POST-ит webhook. Мы пытаемся отправить в Kafka. Kafka лежит.

  → producer.Send() вернул ошибку (timeout / connection refused)
  → отвечаем HTTP 500 (или 503 Service Unavailable)
  → провайдер получил 500 → ставит в свою retry-очередь
  → провайдер ретраит через 5-30 сек (зависит от их настроек)

Сколько провайдер будет ретраить:
  По контракту: обычно 3-5 попыток, потом дропает.
  Если Kafka лежит > 2-3 мин → события от Omnicomm теряются на стороне провайдера.

Что делать:
  → алерт "Kafka unavailable" → инцидент → DevOps поднимает Kafka
  → SLA: Kafka должен быть доступен 99.9% (downtime < 8.7 часов/год)
  → при типичном простое (секунды-минуты) — провайдер успевает ретраить
```

#### Burst (аномально много webhook-ов)

```
Нормальный режим: ~400 events/sec от Omnicomm (доля провайдера от общего потока).
Аномалия: 3000 events/sec (баг провайдера, replay старых данных).

Защита — rate limiter на HTTP server:

  limiter := rate.NewLimiter(1000, 2000) // 1000 req/sec, burst 2000

  func handler(w http.ResponseWriter, r *http.Request) {
      if !limiter.Allow() {
          w.WriteHeader(429) // Too Many Requests
          metrics.RateLimited.Inc()
          return
      }
      // ... process
  }

Почему 1000, а не 400:
  Нормальная нагрузка 400 + headroom 150%.
  Покрывает пики (утро, все ТС одновременно стартуют, batch от провайдера после временной недоступности).
  Но не даёт завалить Kafka (Kafka capacity: ~50K/sec на кластер, но лимит на провайдера разумен).

Провайдер получил 429 → ретраит → события не теряются, просто задерживаются.
```

#### Провайдер дублирует webhook (не получил 200 вовремя)

```
Наш response time: 5-10 мс (ждём Kafka ack).
Провайдер ожидает ответ за 3 сек (их timeout).

Если наш ответ ушёл за 3.1 сек (сеть тормозит):
  → провайдер не увидел 200 → ретраит → тот же payload
  → мы получаем дубль
  → дубль проходит в Kafka (адаптер не дедуплицирует — это не его задача)
  → Telematics Service валидирует и роутит в telematics.gps
  → Tracking Service обрабатывает дубль штатно:
    PG: INSERT ... ON CONFLICT (vehicle_id, timestamp) DO NOTHING → noop
    Redis: HSET перезапись тем же значением → noop

Адаптер не дедуплицирует потому что:
  - Stateless (нет кеша, нет Redis) — добавить дедуп = добавить persistent state
  - Дубли корректно обрабатываются downstream через идемпотентные операции
    (ON CONFLICT в PG, HSET overwrite в Redis, ReplacingMergeTree в ClickHouse)
```

#### Провайдер перестал слать (тихий отказ)

```
Webhook — no keepalive. Если Omnicomm лёг, мы просто не получаем POST-ы.
В отличие от WS (где disconnect явный), тут нет сигнала.

Обнаружение:
  Метрика: adapter_last_event_received_timestamp{provider="omnicomm"}
  Алерт: now() - last_event > 5 мин → "No events from Omnicomm > 5 min"

Реакция:
  → Проверяем: Omnicomm лёг, или у нас проблема (наш endpoint недоступен снаружи)?
  → Health check Omnicomm (если есть status endpoint)
  → Если проблема у нас — NGINX Ingress, DNS, TLS cert expired
  → Если проблема у них — эскалация, ждём восстановления
```

#### Body > 1 MB

```
Защита от аномально большого payload (случайный / злонамеренный):

  http.MaxBytesReader(w, r.Body, 1<<20) // 1 MB

  → Если body > 1 MB → read обрывается → 413 Payload Too Large
  → Метрика: adapter_oversized_payload_total{provider="omnicomm"}

Нормальный payload: одно событие ~500 байт. 1 MB = ~2000 событий в одном POST.
Это аномалия — webhook шлёт по одному событию, не батчами.
```

#### TLS-сертификат нашего endpoint истёк

```
cert-manager в K8s автоматически ротирует сертификаты (Let's Encrypt, 90 дней).

Если автоматическая ротация не сработала:
  → Omnicomm получает TLS error при POST → ретраит → те же TLS errors
  → На нашей стороне: 0 входящих запросов → алерт "No events from Omnicomm"
  → Диагностика: curl к нашему endpoint → TLS error → ротируем cert вручную

Метрика: cert_expiry_days{host="omnicomm-webhook.tms.internal"} → алерт за 14 дней до expiry.
```

---

## 4. Wialon Adapter (TCP binary, push)

### Как работает

```
Трекеры Wialon (тысячи устройств)
    │
    │  persistent TCP connections
    │  Wialon IPS protocol (binary)
    │
    ▼
Wialon Adapter (TCP server)
    │
    │  binary frames → protobuf
    │
    ▼
Kafka: telematics.raw
```

**Модель:** мы — TCP-сервер, трекеры подключаются напрямую (через Wialon gateway). Каждый трекер (или gateway) = одно persistent TCP-соединение. Самый сложный адаптер — бинарный протокол, тысячи одновременных соединений, goroutine per connection.

### Протокол Wialon IPS

- **Транспорт:** raw TCP (не HTTP, не WebSocket)
- **Формат:** бинарный, length-prefix framing
- **ACK:** после каждого фрейма сервер отправляет ACK-фрейм обратно
- **Аутентификация:** device_id + password в первом фрейме (login packet)

### Frame Structure

```
┌──────────┬──────────┬──────────────────┬──────┐
│ Length    │ Type     │ Payload          │ CRC  │
│ 2 bytes  │ 1 byte   │ variable         │ 2 B  │
│ (uint16) │          │                  │      │
└──────────┴──────────┴──────────────────┴──────┘

Type:
  0x01 = Login (device_id, password)
  0x02 = GPS data
  0x03 = Sensor data (engine)
  0x04 = Alarm (driver event)
  0xFF = Ping

ACK frame (мы → трекер):
  0x06 + original frame type + status byte (0x01 = OK, 0x00 = error)
```

### TCP Server на Go

```go
listener, _ := net.Listen("tcp", ":5001")

for {
    conn, _ := listener.Accept()
    go handleConnection(conn) // goroutine per connection
}

func handleConnection(conn net.Conn) {
    defer conn.Close()
    reader := bufio.NewReaderSize(conn, 4096) // буфер 4 KB на соединение

    // Login phase
    loginFrame, err := readFrame(reader)
    if err != nil || loginFrame.Type != 0x01 {
        return // невалидное соединение
    }
    deviceID := parseLogin(loginFrame)
    sendACK(conn, 0x01, 0x01) // ACK login OK

    // Data phase
    conn.SetReadDeadline(time.Now().Add(60 * time.Second))
    for {
        frame, err := readFrame(reader)
        if err != nil {
            return // disconnect, горутина завершается
        }
        conn.SetReadDeadline(time.Now().Add(60 * time.Second)) // reset deadline

        event := parseFrame(deviceID, frame)
        if event != nil {
            kafkaProducer.Send("telematics.raw", event)
        }
        sendACK(conn, frame.Type, 0x01)
    }
}
```

### Edge Cases

#### Неполный TCP-фрейм (partial read)

```
TCP — stream protocol, не message protocol. Один send() на клиенте не гарантирует
один read() на сервере. Фрейм в 100 байт может прийти как:
  read 1: 50 байт (первая половина)
  read 2: 50 байт (вторая половина)

Или наоборот — два фрейма в одном read:
  read 1: 200 байт (фрейм 1 целиком + начало фрейма 2)

Решение — length-prefix protocol:

  func readFrame(r *bufio.Reader) (*Frame, error) {
      // 1. Читаем длину (2 байта) — ВСЕГДА ждём полные 2 байта
      lengthBuf := make([]byte, 2)
      _, err := io.ReadFull(r, lengthBuf)  // ReadFull, не Read
      if err != nil {
          return nil, err
      }
      length := binary.BigEndian.Uint16(lengthBuf)

      // 2. Читаем payload целиком (length байт)
      payload := make([]byte, length)
      _, err = io.ReadFull(r, payload)  // ReadFull — ждём ВСЕ length байт
      if err != nil {
          return nil, err
      }

      return parsePayload(payload)
  }

io.ReadFull блокируется пока не прочитает ровно N байт (или ошибка / timeout).
bufio.Reader буферизует — минимизирует syscall-ы.
```

#### Битый фрейм (CRC mismatch)

```
Payload прочитан, но CRC не совпадает.

Причины: corruption в сети (редко, но бывает), баг в firmware трекера.

  → Дропаем ЭТОТ фрейм
  → Отправляем ACK с status=0x00 (error) → трекер поймёт что фрейм не принят
  → Метрика: adapter_crc_errors_total{provider="wialon"}
  → Соединение НЕ рвём — следующий фрейм с высокой вероятностью валидный
  → Если 10 CRC errors подряд → закрываем соединение (трекер неисправен)
```

#### TCP connection reset (трекер потерял связь)

```
Трекер едет в тоннель, пропадает сеть, перезагружается.

  → conn.Read() вернул io.EOF или net.ErrClosed
  → горутина handleConnection завершается
  → defer conn.Close() закрывает соединение, освобождает ресурсы
  → метрика: active_tcp_connections gauge -1

Трекер выехал из тоннеля:
  → firmware трекера автоматически реконнектится к нашему TCP server
  → listener.Accept() → новая горутина → новый login → данные пошли

Потеря данных:
  Зависит от firmware. Большинство трекеров буферизуют данные при потере связи
  и досылают после реконнекта. Окно дублей = то что мы приняли, но трекер
  не получил ACK → пришлёт повторно → Tracking отбрасывает через ON CONFLICT
  (vehicle_id, timestamp) DO NOTHING.
```

#### Read deadline timeout (трекер молчит)

```
conn.SetReadDeadline(time.Now().Add(60 * time.Second))

Если трекер не прислал ни одного фрейма за 60 секунд:
  → Read() вернёт os.ErrDeadlineExceeded
  → закрываем соединение, горутина завершается

Зачем:
  Без deadline: трекер подключился и замолчал → горутина висит вечно → goroutine leak.
  С deadline: гарантируем что мёртвые соединения не копятся.

Почему 60 сек:
  - GPS шлётся каждые 1-10 сек (зависит от настройки трекера)
  - Ping фрейм (type=0xFF) раз в 30 сек
  - 60 сек = два пропущенных ping-а = точно мёртвое соединение
```

#### Goroutine leak

```
Каждое TCP-соединение = 1 горутина + 4 KB стека (по умолчанию в Go) + bufio.Reader 4 KB.
~8 KB на соединение × 15 000 соединений = ~120 MB. Нормально.

Проблема: если горутины не завершаются (трекер молчит, нет deadline):
  10 000 зомби-горутин × 8 KB = 80 MB. Через неделю = OOM.

Защита:
  1. Read deadline 60 сек (основная)
  2. Метрика: active_tcp_connections gauge
     → алерт если > 18 000 (15K × 1.2 headroom)
  3. pprof endpoint: /debug/pprof/goroutine → видно количество горутин и где висят
  4. Максимум соединений на listener:

     var connSemaphore = make(chan struct{}, 20000)

     func acceptLoop(listener net.Listener) {
         for {
             conn, _ := listener.Accept()
             select {
             case connSemaphore <- struct{}{}:
                 go func() {
                     defer func() { <-connSemaphore }()
                     handleConnection(conn)
                 }()
             default:
                 conn.Close() // лимит достигнут
                 metrics.ConnectionsRejected.Inc()
             }
         }
     }
```

#### Версионирование протокола

```
Wialon IPS v1 и v2 — разный формат payload при том же framing.

Обнаружение версии:
  Login packet содержит version byte.
  v1: login payload = [device_id][password]
  v2: login payload = [version=0x02][device_id][password][capabilities]

  version := loginFrame.Payload[0]
  switch version {
  case 0x01:
      parser = &WialonV1Parser{}
  case 0x02:
      parser = &WialonV2Parser{}
  default:
      // неизвестная версия → close connection, метрика
  }

Оба парсера реализуют один интерфейс:
  type FrameParser interface {
      Parse(frame *Frame) (*RawTelematicsEvent, error)
  }

При выходе v3: добавляем WialonV3Parser, деплоим. Старые трекеры на v1/v2 продолжают работать.
```

#### Backpressure от Kafka

```
Каждая горутина (= каждое TCP-соединение) шлёт в Kafka после парсинга фрейма.
Kafka producer общий: async, внутренний буфер 10K сообщений.

Kafka тормозит → producer buffer заполняется → Send() блокируется.

Проблема: если Send() заблокировался → горутина не читает TCP →
TCP buffer (kernel, 64 KB) заполняется → трекер не может отправить →
трекер копит в свой буфер → overflow → потеря на стороне трекера.

Решение: Send() с timeout:

  ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
  defer cancel()
  err := kafkaProducer.Send(ctx, "telematics.raw", event)
  if err != nil {
      metrics.KafkaSendDropped.Inc()
      // дроп события, НЕ блокируем TCP read
  }

100 мс timeout: если Kafka не принял за 100 мс — дропаем.
Лучше потерять одно событие чем заблокировать TCP и потерять всё соединение.
```

#### Половина трекеров переехала на другой gateway

```
Wialon работает через gateway (промежуточный сервер).
Если gateway заменили — все трекеры переподключаются одновременно.

  t=0     5000 TCP-соединений разорвались
  t=0-5   5000 горутин завершились, connSemaphore освободился
  t=1-10  5000 новых TCP-соединений от нового gateway
          → 5000 Accept() → 5000 новых горутин
          → 5000 login packets → маппинг device_id → vehicle_id
          → если login burst: listener Accept() справляется (Go net.Listener
            использует epoll/kqueue, не блокируется)

Проблема: 5000 login packets одновременно → 5000 gRPC запросов к Vehicle Service?
Нет — маппинг в памяти. Login только проверяет device_id в локальном cache.
```

---

## 5. АвтоГРАФ Adapter (SOAP XML, pull)

### Как работает

```
АвтоГРАФ SOAP API (legacy)
    ▲
    │  HTTP POST (SOAP XML envelope)
    │  каждые 5 сек
    │
АвтоГРАФ Adapter (SOAP client, poller)
    │
    │  XML → protobuf
    │
    ▼
Kafka: telematics.raw
```

**Модель:** мы — клиент, АвтоГРАФ — сервер. Legacy-провайдер, SOAP API, тяжёлый XML. Поллим раз в 5 сек (а не в 1 сек как СКАУТ) потому что SOAP endpoint медленный и rate limit строже.

### Протокол

- **Транспорт:** HTTPS, SOAP 1.1
- **Формат запроса:** XML envelope
- **Формат ответа:** XML envelope с массивом событий
- **Аутентификация:** WS-Security header (username + password в XML)

### SOAP запрос

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:tel="http://avtograf.ru/telemetry/v1">
  <soapenv:Header>
    <wsse:Security>
      <wsse:UsernameToken>
        <wsse:Username>tms_lanit</wsse:Username>
        <wsse:Password>***</wsse:Password>
      </wsse:UsernameToken>
    </wsse:Security>
  </soapenv:Header>
  <soapenv:Body>
    <tel:GetEventsRequest>
      <tel:FromTimestamp>1704067200000</tel:FromTimestamp>
      <tel:MaxCount>500</tel:MaxCount>
    </tel:GetEventsRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

### SOAP ответ

```xml
<soapenv:Envelope>
  <soapenv:Body>
    <tel:GetEventsResponse>
      <tel:Events>
        <tel:Event>
          <tel:DeviceId>AG-4501</tel:DeviceId>
          <tel:Type>GPS</tel:Type>
          <tel:Timestamp>1704067200000</tel:Timestamp>
          <tel:Latitude>55.751244</tel:Latitude>
          <tel:Longitude>37.618423</tel:Longitude>
          <tel:Speed>82</tel:Speed>
          <tel:Course>45</tel:Course>
          <tel:Altitude>156</tel:Altitude>
          <tel:Satellites>12</tel:Satellites>
        </tel:Event>
        <!-- ... ещё события -->
      </tel:Events>
      <tel:HasMore>false</tel:HasMore>
    </tel:GetEventsResponse>
  </soapenv:Body>
</soapenv:Envelope>
```

### Парсинг XML в Go

```go
type GetEventsResponse struct {
    XMLName xml.Name `xml:"GetEventsResponse"`
    Events  []Event  `xml:"Events>Event"`
    HasMore bool     `xml:"HasMore"`
}

type Event struct {
    DeviceID   string  `xml:"DeviceId"`
    Type       string  `xml:"Type"`
    Timestamp  int64   `xml:"Timestamp"`
    Latitude   float64 `xml:"Latitude"`
    Longitude  float64 `xml:"Longitude"`
    Speed      float64 `xml:"Speed"`
    Course     float64 `xml:"Course"`
    Altitude   float64 `xml:"Altitude"`
    Satellites int32   `xml:"Satellites"`
}
```

### Почему poll раз в 5 сек, а не в 1 сек

```
1. SOAP endpoint медленный: response time 500 мс - 2 сек (XML serialization тяжёлая)
2. Rate limit провайдера: 20 req/мин = 1 запрос в 3 сек минимум
3. Нагрузка от АвтоГРАФ самая маленькая (legacy, мало ТС на этом провайдере)
4. 5 сек — с запасом, не упираемся в rate limit, ответ успевает прийти

Задержка доставки GPS от АвтоГРАФ: до 5 сек (poll interval).
Для GPS в логистике — допустимо (диспетчер не заметит 5 сек).
```

### Edge Cases

#### Невалидный XML в ответе

```
Причины: баг провайдера, прокси порезал ответ, encoding issue.

  → xml.Unmarshal() вернул ошибку
  → дропаем ВЕСЬ батч (не отдельное событие — XML broken целиком)
  → метрика: adapter_parse_errors_total{provider="avtograf"}
  → lastTimestamp НЕ обновляем → при следующем poll запросим те же данные
  → алерт: > 3 parse errors подряд → систематическая проблема

Отличие от JSON-адаптеров: JSON можно парсить по строкам (одно битое событие
не ломает остальные). XML — дерево, одна битая скобка ломает весь документ.
```

#### SOAP Fault

```
Стандартный механизм ошибок в SOAP:

<soapenv:Fault>
  <faultcode>Server</faultcode>
  <faultstring>Internal error</faultstring>
</soapenv:Fault>

  → Логируем faultcode + faultstring
  → Retry с backoff (аналогично HTTP 500)
  → lastTimestamp не обновляем
  → Метрика: adapter_soap_faults_total{provider="avtograf", code="Server"}

SOAP Fault != HTTP 500. HTTP может быть 200, но внутри — Fault.
Проверяем ОРИГНАЛ XML на наличие Fault ПЕРЕД парсингом событий.
```

#### Кодировка windows-1251

```
Legacy-система → XML может быть в windows-1251 вместо UTF-8.

Проблема: Go xml.Decoder ожидает UTF-8. Кириллица в windows-1251 → мусор или ошибка парсинга.

Определение кодировки:
  <?xml version="1.0" encoding="windows-1251"?>

Решение:
  import "golang.org/x/text/encoding/charmap"

  func decodeResponse(body io.Reader) io.Reader {
      // Читаем первые байты для определения encoding
      buf := bufio.NewReader(body)
      header, _ := buf.Peek(100)

      if bytes.Contains(header, []byte("windows-1251")) {
          return charmap.Windows1251.NewDecoder().Reader(buf)
      }
      return buf // UTF-8 по умолчанию
  }

  // Использование:
  reader := decodeResponse(resp.Body)
  xml.NewDecoder(reader).Decode(&response)
```

#### Провайдер вернул HTML вместо XML (502 от reverse proxy)

```
Провайдер за NGINX/Apache. NGINX вернул свою 502 Bad Gateway page (HTML).

  → xml.Unmarshal() на HTML → ошибка
  → Но лучше поймать раньше:

  contentType := resp.Header.Get("Content-Type")
  if !strings.Contains(contentType, "text/xml") &&
     !strings.Contains(contentType, "application/xml") {
      // не XML → дроп + алерт
      metrics.UnexpectedContentType.With("provider", "avtograf").Inc()
      return
  }

  → Дополнительно: если HTTP status != 200 → не парсим body, сразу retry
```

#### Большой батч после простоя

```
Адаптер был выключен 30 минут. Провайдер накопил 30 мин × ~400 events/sec (доля АвтоГРАФ) = ~720K событий.

Запрашиваем MaxCount=500 → получаем 500 + HasMore=true.
Следующий запрос: from=lastTs → ещё 500. И так ~1440 раз.

Проблема: 1440 × 2 сек (SOAP latency) = ~48 минут на catch-up. Одним потоком.

Решение: при catch-up mode параллелим запросы:

  if hasMore {
      // catch-up: 3 параллельных запроса с разными offset-ами
      // (если API поддерживает offset/page)
      // если не поддерживает — последовательно, но без sleep между запросами
  }

Также: Kafka producer в batch mode — чанки по 500 событий → один batch Send.
Не по одному, иначе ~720K individual Send() = перегруз.
```

#### Self-signed сертификат провайдера

```
Legacy-провайдер → HTTPS с self-signed сертификатом.

НЕПРАВИЛЬНО:
  &tls.Config{InsecureSkipVerify: true}  // отключает ВСЮ проверку TLS → MITM

ПРАВИЛЬНО:
  // Загружаем CA-сертификат провайдера
  caCert, _ := os.ReadFile("/etc/ssl/certs/avtograf-ca.pem")
  caCertPool := x509.NewCertPool()
  caCertPool.AppendCertsFromPEM(caCert)

  client := &http.Client{
      Transport: &http.Transport{
          TLSClientConfig: &tls.Config{
              RootCAs: caCertPool, // доверяем только этому CA
          },
      },
  }

CA-сертификат — в K8s Secret, монтируется в под.
При ротации: обновляем Secret → rolling restart.
```

#### Медленный ответ (SOAP timeout)

```
Timeout: 10 сек (SOAP бывает тяжёлый).

  ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
  defer cancel()
  resp, err := client.Do(req.WithContext(ctx))

Почему 10 сек (а не 5 как у СКАУТ):
  - SOAP XML serialization/deserialization на стороне провайдера тяжелее JSON
  - Нормальный response time: 500 мс - 2 сек
  - 10 сек = coverage для медленных запросов с большим батчем

Если timeout:
  → retry с backoff
  → lastTimestamp не обновляем
  → 3 timeout подряд → circuit breaker OPEN
```

---

## Circuit Breaker (общий для всех адаптеров)

### Зачем

Без circuit breaker: провайдер лежит → адаптер долбит его запросами каждую секунду → бессмысленно, спамим логи, тратим ресурсы, можем ещё сильнее нагрузить больного провайдера.

С circuit breaker: 5 ошибок подряд → перестаём стучаться → ждём → пробуем одним запросом → работает? → возобновляем.

### State Machine

```
          5 ошибок за 30 сек
CLOSED ─────────────────────────→ OPEN
  ▲                                 │
  │                                 │ 30 сек прошло
  │         1 успешный probe        ▼
  └──────────────────────────── HALF-OPEN
                                    │
                   probe failed     │
                   ─────────────────┘ (обратно в OPEN)
```

### Параметры

| Параметр | Значение | Почему |
|---|---|---|
| Порог ошибок | 5 за 30 сек | Одна ошибка — случайность. 5 — тренд |
| Timeout (OPEN → HALF-OPEN) | 30 сек | Даём провайдеру время восстановиться |
| Probe requests | 1 | Минимальная нагрузка для проверки |
| Success threshold (HALF-OPEN → CLOSED) | 1 успешный probe | Провайдер жив → возобновляем |

### Метрики

```
adapter_circuit_breaker_state{provider="navixy"}    // 0=closed, 1=open, 2=half-open
adapter_circuit_breaker_trips_total{provider="navixy"}  // сколько раз открывался
```

### Retry budget поверх CB

Circuit Breaker сам по себе не спасает от "вечного цикла проб" при долгом инциденте: каждые 30 сек HALF-OPEN пропускает probe, probe падает → снова OPEN → через 30 сек снова probe → и так часами. Если у провайдера реальный инцидент на часы — мы всё равно забрасываем его probe-запросами каждые 30 сек.

Поверх CB добавлен **retry budget** — token bucket на ретраи: **100 попыток в минуту на провайдера**. Если ретраи исчерпаны — адаптер уходит в extended cooldown на 5 мин (вместо стандартных 30 сек half-open). Это защищает:

- **Нас** — не тратим CPU и сетевой стек на гарантированно мёртвые вызовы, не спамим логи.
- **Провайдера** — не DDoS-им его восстанавливающийся сервис. Когда он поднимается после инцидента, первое что ему не нужно — это 5 наших адаптеров одновременно лупящих probe запросы.

```
adapter_retry_budget_remaining{provider="navixy"}   // сколько попыток осталось
adapter_extended_cooldown_active{provider="navixy"} // 1 = в extended cooldown
```

---

## Kafka Producer (общий для всех адаптеров)

### Конфигурация

```go
producer := kafka.NewAsyncProducer(config{
    Brokers:        []string{"kafka-0:9092", "kafka-1:9092", "kafka-2:9092"},
    Topic:          "telematics.raw",
    Acks:           kafka.WaitForAll,    // acks=all: ждём подтверждения от всех ISR
    MaxRetries:     3,
    RetryBackoff:   100 * time.Millisecond,
    BatchSize:      10000,               // буфер 10K сообщений
    FlushInterval:  100 * time.Millisecond,
    Compression:    kafka.Snappy,        // ~30% экономия на сети
    KeyFunc:        func(e) string { return e.VehicleID }, // партиционирование по vehicle_id
})
```

### Async producer

```
Почему async, а не sync:

Sync producer:
  Send() → ждём ack от Kafka → return
  Latency: 1-5 мс на сообщение
  2K events/sec × 3 мс = часть горутин копится в ожидании — неоптимально

Async producer:
  Send() → кладём в буфер → return мгновенно
  Фоновая горутина: буфер → batch → Kafka
  Latency Send(): наносекунды

Цена async: если адаптер упал до flush → потеря буфера.
При 100 мс flush interval → максимум потеря 100 мс × ~400 events/sec (на один адаптер) = ~40 событий.
Pull-адаптеры заберут повторно. Push-адаптеры — провайдер дошлёт.
```

### `acks=all` vs `acks=1`

```
acks=1:  лидер записал → ack. Если лидер упал до репликации → потеря.
acks=all: лидер + все ISR записали → ack. Потеря только при падении ВСЕХ реплик.

Мы используем acks=all потому что:
  - Kafka — единственное место где данные живут до обработки
  - Если потеряли сообщение в Kafka — потеряли навсегда (адаптер stateless, уже забыл)
  - Latency разница: +1-2 мс. При async producer — незаметно

С min.insync.replicas=2 и replication.factor=3:
  acks=all = ждём подтверждения от 2 из 3 брокеров. Один может быть медленным.
```

---

## Метрики (общие для всех адаптеров)

```
# Сколько событий обработано (по провайдеру и статусу)
adapter_events_total{provider="navixy", status="success|dropped|error"}

# Ошибки по типу
adapter_errors_total{provider="navixy", error_type="parse|auth|timeout|kafka_send"}

# Latency обработки (от получения до отправки в Kafka)
adapter_processing_duration_seconds{provider="navixy"}

# Kafka producer
adapter_kafka_send_total{provider="navixy", status="success|error"}
adapter_kafka_buffer_size{provider="navixy"}  # текущий размер буфера

# Circuit breaker
adapter_circuit_breaker_state{provider="navixy"}  # 0/1/2
adapter_circuit_breaker_trips_total{provider="navixy"}

# Provider-specific
adapter_ws_disconnect_total{provider="navixy"}
adapter_http_errors_total{provider="scout", status="429|500|502|503"}
adapter_active_tcp_connections{provider="wialon"}  # gauge
adapter_crc_errors_total{provider="wialon"}
adapter_unknown_device_total{provider="navixy"}

# Лаг доставки (received_at - timestamp)
adapter_delivery_lag_seconds{provider="navixy"}
```

### Алерты

| Алерт | Условие | Severity |
|---|---|---|
| Adapter Down | circuit breaker OPEN > 2 мин | Critical |
| High Drop Rate | events_dropped > 100/sec > 1 мин | Warning |
| Parse Errors | parse_errors > 10/мин | Warning |
| Unknown Devices | unknown_device > 100/мин | Warning |
| Delivery Lag | delivery_lag > 30 сек | Warning |
| Kafka Send Errors | kafka_send_errors > 0 > 1 мин | Critical |
| TCP Connections | active_tcp_connections > 18K | Warning |
| No Events | no events from provider > 5 мин | Critical |

---

## Сводная таблица edge cases

| Сценарий | Navixy (WS) | СКАУТ (REST) | Omnicomm (Webhook) | Wialon (TCP) | АвтоГРАФ (SOAP) |
|---|---|---|---|---|---|
| **Провайдер лёг** | Reconnect backoff → CB open | Retry backoff → CB open | Нет событий → алерт | Трекеры реконнектятся | Retry backoff → CB open |
| **Сеть между нами** | WS disconnect → reconnect | Timeout → retry | Timeout → провайдер retries | TCP reset → трекер reconnect | Timeout → retry |
| **Невалидные данные** | Дроп фрейма, stream живёт | Дроп события | 400 Bad Request | Дроп фрейма, TCP живёт | Дроп всего батча (XML) |
| **Kafka лежит** | Буфер → дроп старых | Не обновляем lastTs | 500 → провайдер retries | Send timeout → дроп | Не обновляем lastTs |
| **Адаптер рестарт** | WS reconnect, провайдер досылает | from=now-30s, дубли ok | Провайдер retries pending | Трекеры reconnect | from=now-30s, дубли ok |
| **Burst нагрузки** | Channel cap → дроп | Пагинация, catch-up | Rate limiter 429 | Conn semaphore | Чанки по 500 |
| **Потеря данных** | Только при disconnect > 60s | Нет (pull) | Только при Kafka > 3 мин down | Зависит от firmware буфера | Нет (pull) |

---

## Ключевые решения

| Решение | Почему | Trade-off |
|---|---|---|
| 1 провайдер = 1 сервис | Изоляция отказов. Wialon лёг → остальные 4 работают | 5 деплойментов вместо 1. Но каждый маленький (20 МБ binary) |
| Адаптеры stateless | Нет PG/Redis = нет миграций, нет бэкапов, нет точек отказа. При рестарте — чистый лист | При рестарте pull-адаптеров: overlap 30 сек, ~12K дублей на адаптер. Downstream идемпотентен (ON CONFLICT в Tracking) |
| Дедупликация НЕ в адаптере и НЕ в Telematics | Downstream consumer-ы уже идемпотентны (ON CONFLICT, HSET overwrite, ReplacingMergeTree, event_id key). Отдельный слой дедупа избыточен и всё равно не гарантирует exactly-once | Дубли проходят в Kafka raw. Чуть больше write IO в PG, но архитектура чище |
| Async Kafka producer | Наносекунды latency Send(). Не блокируем receiver | Потеря буфера при crash (~300 events). Допустимо |
| acks=all | Kafka — единственное хранилище до обработки. Потеря = навсегда | +1-2 мс latency (незаметно с async producer) |
| Circuit breaker | Не спамим мёртвого провайдера. Быстрое обнаружение проблем | 30 сек без данных при CB open. Для GPS — не критично |
| Маппинг device→vehicle в памяти | ~2K events/sec × 5 адаптеров → лишние gRPC hops на каждое событие = +latency, нагрузка на Vehicle Service | Маппинг может быть stale до прихода Kafka event обновления парка. Обычно секунды |
| Read deadline 60s (TCP) | Защита от goroutine leak | Трекер в длинном тоннеле (>60s) → disconnect → reconnect. Мелочь |

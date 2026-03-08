- **Без таймаутов** = горутина висит вечно, fd утекает, коннекты кончаются, сервис деградирует каскадно. Таймаут — **обязателен** на каждом HTTP-вызове
- **Go http.Client**: `Timeout` (общий, от запроса до конца чтения body), `DialTimeout` (TCP connect), `TLSHandshakeTimeout`, `ResponseHeaderTimeout` (до первого байта ответа), `IdleConnTimeout`
- **Go http.Server**: `ReadTimeout` (от accept до конца чтения body), `WriteTimeout` (от конца чтения до конца записи ответа), `IdleTimeout` (keep-alive между запросами), `ReadHeaderTimeout`
- Подбор: **p99 латенси upstream × 2-3** как стартовая точка. Долгие операции → отдельный клиент с большим timeout. Timeout **всегда меньше** чем у вызывающего сервиса (иначе каскадный timeout)

---

## Зачем

```
Без таймаута:
  1. Upstream завис → горутина ждёт бесконечно
  2. Горутина держит fd → fd не освобождается
  3. Новые запросы → новые горутины → все висят
  4. fd закончились → сервис не принимает запросы
  5. Upstream сервис тоже копит ожидающих → каскадная деградация
```

Один зависший upstream без таймаутов **убивает всю цепочку** микросервисов.

## Go http.Client таймауты

```go
client := &http.Client{
    Timeout: 5 * time.Second,  // общий таймаут (от запроса до конца чтения body)
    Transport: &http.Transport{
        DialContext: (&net.Dialer{
            Timeout:   1 * time.Second,  // TCP connect
            KeepAlive: 30 * time.Second, // TCP keep-alive probes
        }).DialContext,
        TLSHandshakeTimeout:   1 * time.Second,   // TLS handshake
        ResponseHeaderTimeout: 3 * time.Second,    // до первого байта ответа
        IdleConnTimeout:       90 * time.Second,   // idle keep-alive conn
        MaxIdleConns:          100,                 // пул соединений
        MaxIdleConnsPerHost:   10,
    },
}
```

```
Время ──────────────────────────────────────────────────→
|--Dial--|--TLS--|--Send--|--Wait--|--Headers--|--Body--|
|  1s    |  1s   |        | ResponseHeader 3s |        |
|──────────────── Timeout 5s ─────────────────────────|
```

**Timeout** — **общий**: включает всё от начала до конца чтения body. Если body большой (скачивание файла) → нужен context с deadline вместо Timeout.

### Context вместо общего Timeout

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
resp, err := client.Do(req)
// context.DeadlineExceeded если превысили
```

**Преимущество**: context прокидывается по цепочке → отмена каскадно.

## Go http.Server таймауты

```go
srv := &http.Server{
    Addr:              ":8080",
    ReadTimeout:       5 * time.Second,   // от accept до конца чтения body
    ReadHeaderTimeout: 2 * time.Second,   // от accept до конца чтения headers
    WriteTimeout:      10 * time.Second,  // от конца чтения до конца записи ответа
    IdleTimeout:       120 * time.Second, // между keep-alive запросами
    MaxHeaderBytes:    1 << 20,           // 1 MB
}
```

```
Время ──────────────────────────────────────────→
|--Accept--|--Headers--|--Body--|--Process--|--Write--|--Idle--|
|     ReadHeaderTimeout 2s     |                              |
|──────── ReadTimeout 5s ──────|                              |
                                |──── WriteTimeout 10s ───────|
                                                    |IdleTimeout|
```

**Без ReadTimeout**: slowloris attack — клиент шлёт по 1 байту в секунду → горутина висит вечно → исчерпание ресурсов.

**Без WriteTimeout**: медленный клиент не читает ответ → горутина висит на Write → утечка.

## Как подобрать

**Стартовая точка**: `p99 латенси upstream × 2-3`.

```
Upstream p99 = 200ms → timeout = 500ms-600ms
Upstream p99 = 2s    → timeout = 5s-6s
```

**Правила:**
- Timeout клиента **< timeout вызывающего** (A вызывает B: timeout_A < timeout_caller_of_A)
- Разные upstream'ы → **разные клиенты** с разными timeout'ами
- Долгие операции (загрузка файла, отчёт) → **отдельный клиент** или context с большим deadline

```go
// ПЛОХО: один клиент на всё
var client = &http.Client{Timeout: 30 * time.Second}  // долгий для всех

// ХОРОШО: разные клиенты
var fastClient = &http.Client{Timeout: 1 * time.Second}   // для лёгких запросов
var slowClient = &http.Client{Timeout: 30 * time.Second}  // для загрузки файлов
```

## Дефолты Go (ловушка!)

```go
// http.DefaultClient — НУЛЕВОЙ timeout!
resp, err := http.Get("https://example.com")  // может висеть БЕСКОНЕЧНО
```

**Никогда** не используй `http.DefaultClient` в проде. Всегда создавай клиент с таймаутами.

```go
// http.DefaultTransport — тоже нет ResponseHeaderTimeout
// MaxIdleConnsPerHost = 2 (маленький пул!)
```

## Связь
- [[HTTP протокол и версии]] — keep-alive, connection management
- [[TCP — flow и congestion control]] — TCP keep-alive vs HTTP idle timeout
- [[TCP — соединение]] — TCP handshake timeout = DialTimeout

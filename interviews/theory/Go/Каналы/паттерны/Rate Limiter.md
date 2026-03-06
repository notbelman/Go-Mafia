- Rate Limiter — ограничение количества операций в единицу времени
- Leaky Bucket на каналах: буферизированный канал = ведро, тикер вычитывает = утечка
- Allow() пытается записать в канал: есть место → true, нет → false (лимит превышен)

---

## Идея Leaky Bucket

Ведро с дыркой: вода (запросы) наливается сверху, вытекает снизу с постоянной скоростью. ^rl-leaky-idea

```
    ┌─────────┐  ← запросы (Allow)
    │ ░░░░░░░ │  ← ведро (буферизированный канал)
    │ ░░░░░░░ │     размер = лимит
    └────●────┘  ← утечка (тикер читает из канала)
```

- Ведро полное → новые запросы отклоняются
- Тикер регулярно "вычерпывает" из ведра → освобождает место

## Реализация

```go
type RateLimiter struct {
    bucket chan struct{}
}

func NewRateLimiter(limit int, period time.Duration) *RateLimiter {
    rl := &RateLimiter{
        bucket: make(chan struct{}, limit),
    }

    leakInterval := period / time.Duration(limit)

    // заполняем ведро
    for i := 0; i < limit; i++ {
        rl.bucket <- struct{}{}
    }

    // горутина-утечка
    go rl.leak(leakInterval)
    return rl
}

func (rl *RateLimiter) leak(interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for range ticker.C {
        select {
        case <-rl.bucket:  // вычерпнуть одну единицу
        default:            // ведро пустое — ничего не делаем
        }
    }
}

func (rl *RateLimiter) Allow() bool {
    select {
    case rl.bucket <- struct{}{}:
        return true   // есть место → разрешаем
    default:
        return false  // ведро полное → отклоняем
    }
}
```

^rl-impl

## Как работает

1. Ведро (канал) размером `limit` — сколько запросов можно принять
2. `Allow()` пишет в канал: если буфер не полон → true. Полон → false
3. Тикер каждые `period/limit` читает из канала → освобождает место для новых запросов

Пример: limit=5, period=1s → тикер читает каждые 200ms → 5 req/sec. ^rl-how

## Инвертированная семантика

В обычном Token Bucket разрешение = наличие токена. Здесь **инвертировано**: разрешение = **место в ведре** (можно записать). Ведро заполнено = лимит исчерпан. Тикер освобождает место = добавляет "ёмкость". ^rl-inverted

## Другие алгоритмы

Leaky Bucket — один из нескольких: ^rl-algorithms

- **Token Bucket** — токены генерируются с постоянной скоростью, разрешает burst
- **Fixed Window** — счётчик обнуляется каждые N секунд
- **Sliding Window** — скользящее окно, точнее чем fixed

Leaky Bucket на каналах — самый простой для Go. ^rl-leaky-simplest

## Связь
- [[Семафор]] — семафор ограничивает параллелизм, rate limiter — частоту
- [[Ticker]] — тикер как механизм утечки

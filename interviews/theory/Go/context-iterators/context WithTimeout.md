- автоотмена через указанную длительность (относительное время) ^timeout-relative
- внутри = обёртка над WithDeadline: `WithDeadline(parent, time.Now().Add(timeout))` ^timeout-internals
- ВСЕГДА defer cancel() — даже с таймаутом (защита от утечки горутины при огромном/ошибочном таймауте) ^timeout-defer-cancel

---

## Базовый паттерн

```go
ctx, cancel := context.WithTimeout(parent, 2*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
resp, err := http.DefaultClient.Do(req)  // отменится через 2с
```

^timeout-http-pattern

Библиотека сама следит за ctx.Done(). В стандартной библиотеке это всегда работает корректно. ^timeout-stdlib-correct

В сторонних библиотеках — проверяйте: бывают баги когда сторонняя библиотека не полностью отменяет операции. ^timeout-thirdparty-risk

## Почему defer cancel() даже с таймаутом

```go
// Кажется: "таймаут и так отменит через 5с, зачем cancel?"
ctx, cancel := context.WithTimeout(parent, 5*time.Second)
// А если по ошибке поставили 100 * time.Hour?
// Без defer cancel() — горутина висит 100 часов

// Поэтому ВСЕГДА:
defer cancel()
```

^timeout-wrong-duration-risk

Под капотом WithTimeout создаёт таймер. defer cancel() останавливает таймер и освобождает ресурсы сразу, а не через duration. Линтеры явно требуют defer cancel(). ^timeout-timer-resources

## Проверка причины отмены

```go
select {
case <-ctx.Done():
    if ctx.Err() == context.DeadlineExceeded {
        // таймаут
    }
    if ctx.Err() == context.Canceled {
        // ручная отмена или отмена родителя
    }
}
```

^timeout-check-cause

## WithTimeoutCause (Go 1.21+)

```go
ctx, cancel := context.WithTimeoutCause(parent, time.Second,
    errors.New("DB query too slow"))

// Если сработал таймаут:
context.Cause(ctx)  // "DB query too slow"

// Если вызвали cancel() явно:
context.Cause(ctx)  // context.Canceled
```

^timeout-cause-example

WithTimeoutCause позволяет задать кастомное сообщение ошибки при таймауте. Если же таймаут не сработал и cancel() вызван явно — Cause возвращает context.Canceled. ^timeout-cause-behavior

С какой версии Go доступен WithTimeoutCause: Go 1.21+. ^timeout-cause-version

## Связь
- [[context WithDeadline]] — абсолютное время вместо длительности
- [[context WithCancel]] — ручная отмена без таймера
- [[Оборачивание функций без контекста]] — когда библиотека не умеет работать с ctx

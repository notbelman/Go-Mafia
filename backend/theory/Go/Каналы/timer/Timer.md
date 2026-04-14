- Timer — одноразовый таймаут: тикнет ровно один раз через N, пишет в канал `.C`
- Основное применение: таймаут операции через select
- GC собирает сам после срабатывания (не нужен Stop для очистки)

---

## Что это

`time.NewTimer(d)` — сработает один раз через `d`: ^timer-what

```go
timer := time.NewTimer(5 * time.Second)
defer timer.Stop()

select {
case <-timer.C:
    fmt.Println("timeout!")
case data := <-dataCh:
    fmt.Println("got data", data)
}
```

После срабатывания канал `.C` закрыт для дальнейших событий — Timer отработал. ^timer-after-fire

## Ticker vs Timer

```
              Ticker                Timer
Сколько раз   бесконечно            один раз
Назначение    периодические события таймаут
Аналогия      будильник каждый час  будильник на 7:00
Пример        heartbeat             таймаут запроса
```
^ticker-vs-timer

## Практический паттерн — таймаут запроса

```go
func requestWithTimeout(timeout time.Duration) (string, error) {
    resultCh := make(chan string, 1)
    go func() {
        // долгий запрос к сервису
        result := doRemoteCall()
        resultCh <- result
    }()

    select {
    case res := <-resultCh:
        return res, nil
    case <-time.After(timeout):
        return "", errors.New("timeout")
    }
}
```

Либо данные пришли, либо таймаут. Без sleep, без busy waiting. ^timer-timeout-pattern

С контекстами (`context.WithTimeout`) это делается проще, но механика та же — select + канал. ^timer-vs-context

## Связь
- [[Ticker]] — периодический вариант
- [[time Tick и time After]] — функция-обёртка time.After
- [[Ticker и Timer утечки и GC]] — GC поведение

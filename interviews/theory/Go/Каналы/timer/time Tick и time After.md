- `time.Tick(d)` и `time.After(d)` — функции-обёртки, возвращают только канал (без доступа к Stop/Reset)
- Ловушка: в select-цикле на **каждой итерации** создаётся новый Ticker/Timer → таймаут никогда не наступит
- Правило: в циклах используй `NewTicker`/`NewTimer` до цикла, в одноразовых — обёртки ок

---

## Что возвращают

```go
// time.Tick — возвращает <-chan Time (только канал)
ch := time.Tick(1 * time.Second)

// time.After — возвращает <-chan Time (один тик)
ch := time.After(5 * time.Second)
```

Удобно, но нет доступа к `.Stop()` и `.Reset()`. ^tick-after-no-stop

## Ловушка в select-цикле

```go
// ОШИБКА — таймаут никогда не сработает!
for {
    select {
    case <-time.After(5 * time.Second):  // новый таймер КАЖДУЮ итерацию
        fmt.Println("timeout")
        return
    case <-time.Tick(1 * time.Second):   // новый тикер КАЖДУЮ итерацию
        fmt.Println("tick")
    }
}
```

Каждый проход цикла:
1. `time.After(5s)` → создаётся **новый** Timer с 5с
2. Тик приходит через 1с → переход на следующую итерацию
3. Старый Timer с оставшимися 4с **выбрасывается**
4. Создаётся новый Timer с 5с → и так навсегда

Таймаут **никогда** не наступит, потому что каждую секунду он обнуляется. ^tick-after-loop-trap

## Правильно

```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()
timer := time.NewTimer(5 * time.Second)
defer timer.Stop()

for {
    select {
    case <-timer.C:
        fmt.Println("timeout")
        return
    case <-ticker.C:
        fmt.Println("tick")
    }
}
```

Ticker и Timer создаются **до** цикла, один раз. Таймаут сработает через 5 секунд. ^tick-after-loop-fix

## Когда обёртки безопасны

`time.After` в select **без цикла** — ок (одноразовый таймаут). ^tick-after-safe-after

`time.Tick` в main-горутине которая живёт вечно — ок (не нужен Stop). ^tick-after-safe-tick

В тестах для простоты — также ок. ^tick-after-safe-tests

## Связь
- [[Ticker]] — NewTicker с доступом к Stop/Reset
- [[Timer]] — NewTimer с доступом к Stop/Reset
- [[Ticker и Timer утечки и GC]] — почему утекали старые тикеры

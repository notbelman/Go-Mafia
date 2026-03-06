- Ticker — периодический таймер: тикает каждые N, пишет событие в канал `.C`
- Внутри: горутина + sleep + запись в буферизированный канал(1)
- С Go 1.23 Stop не обязателен для GC, но хорошая практика

---

## Что это

`time.NewTicker(d)` возвращает структуру с каналом `.C`. Каждые `d` в канал приходит событие `time.Time`. ^ticker-what

```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()

for {
    select {
    case t := <-ticker.C:
        fmt.Println("tick", t)
    case data := <-dataCh:
        fmt.Println("data", data)
        return
    }
}
```

Ключевой паттерн: **select** — либо тик, либо данные из канала. Не sleep, потому что sleep блокирует и мы пропустим данные. ^ticker-select-pattern

## Наивная реализация

```go
type MyTicker struct {
    C        chan time.Time
    interval time.Duration
    stopped  bool
}

func NewMyTicker(d time.Duration) *MyTicker {
    t := &MyTicker{
        C:        make(chan time.Time, 1), // буфер 1
        interval: d,
    }
    go func() {
        for !t.stopped {
            time.Sleep(t.interval)
            t.C <- time.Now() // блокируется если никто не читает
        }
    }()
    return t
}
```

Буфер 1 — чтобы один тик мог накопиться. Если никто не читает — горутина блокируется на записи. ^ticker-buf1

Внутри реального `time.NewTicker` также запускается горутина, которая периодически пишет в буферизированный канал с размером 1. ^ticker-internals

## Когда использовать

Ticker применяется для: периодического flush на диск, heartbeat каждые N секунд, polling с интервалом, любого "делай что-то каждые X". ^ticker-usecases

## Связь
- [[Timer]] — одноразовый вариант (тикнет один раз)
- [[time Tick и time After]] — функции-обёртки и ловушка в цикле
- [[Ticker и Timer утечки и GC]] — почему раньше Stop был обязателен

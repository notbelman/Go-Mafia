---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - HTTP
  - Concurrency
  - Caching
title: AI Weather - высоконагруженный HTTP-сервер
---

## Условие
```go
package main

import (
    "fmt"
    "math/rand"
    "net/http"
    "time"
)

// aiWeatherForecast через нейронную сеть вычисляет прогноз погоды за ~1 секунду
func aiWeatherForecast() int {
    time.Sleep(1 * time.Second)
    return rand.Intn(70) - 30
}

func main() {
    // 10k rps
    http.HandleFunc("/weather", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "{\"temperature\":%d}\n", aiWeatherForecast())
    })
    if err := http.ListenAndServe(":3333", nil); err != nil {
        panic(err)
    }
}
```

**Вопрос:** Что произойдёт, если отправить 10 000 RPS на ручку `/weather`?

**Ответ:** Каждый HTTP-запрос обрабатывается в отдельной горутине. При 10 000 RPS и времени обработки 1 секунда будет одновременно висеть до 10 000 горутин. Это приведёт к росту потребления памяти, нагрузке на планировщик и деградации производительности или краху сервера (OOM, таймауты, ошибки соединения).

## Решение

Вычислять прогноз в фоне раз в N секунд и отдавать клиентам готовое значение:

```go
package main

import (
    "fmt"
    "math/rand"
    "net/http"
    "sync"
    "time"
)

var (
    temperature int
    mu          sync.RWMutex
)

func getWeather(w http.ResponseWriter, r *http.Request) {
    mu.RLock()
    defer mu.RUnlock()
    fmt.Fprintf(w, "{\"temperature\":%d}\n", temperature)
}

func startWeatherUpdater(interval time.Duration) {
    go func() {
        for {
            time.Sleep(interval)
            updateTemperature()
        }
    }()
}

func updateTemperature() {
    mu.Lock()
    defer mu.Unlock()
    temperature = rand.Intn(70) - 30
}

func main() {
    rand.Seed(time.Now().UnixNano())
    updateTemperature()                  // прогрев кэша
    startWeatherUpdater(1 * time.Second) // фоновое обновление

    http.HandleFunc("/weather", getWeather)
    if err := http.ListenAndServe(":3333", nil); err != nil {
        panic(err)
    }
}
```

## Дополнительные вопросы

**Какой примерно максимальный RPS выдержит этот сервис?**

Лимит RPS зависит не от кода — он очень лёгкий: чтение int под `RLock` и запись строки в ответ. Ограничения будут на уровне среды выполнения:
- `GOMAXPROCS` — сколько потоков рантайм может использовать
- Количество ядер CPU
- Объём оперативной памяти
- Пропускная способность сети

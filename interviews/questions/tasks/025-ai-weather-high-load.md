---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - HTTP
  - Concurrency
  - Caching
title: AI Weather - высокая нагрузка на медленный эндпоинт
---

## Условие
```go
package main

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

**Вопрос:** Что произойдёт, если отправить 10 000 RPS на ручку `/weather`, где каждый запрос вызывает `aiWeatherForecast`, работающий ~1 секунду?

**Ответ:** Каждый HTTP-запрос обрабатывается в отдельной горутине, и если `aiWeatherForecast` занимает 1 секунду, то при 10 000 запросах в секунду будет одновременно висеть до 10 000 горутин. Это приведёт к быстрому росту потребления памяти, нагрузке на планировщик и в итоге — к деградации производительности или краху сервера (OOM, таймауты, ошибки соединения).

**Вопрос:** Как исправить?

**Ответ:** Правильный подход — вычислять прогноз в фоне раз в N секунд и отдавать клиентам готовое значение:

## Решение

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

// Обновляет прогноз раз в interval
func startWeatherUpdater(interval time.Duration) {
    go func() {
        for {
            time.Sleep(interval)
            updateTemperature()
        }
    }()
}

// Инициализирует начальное значение прогноза
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

**Вопрос:** Какой примерно максимальный RPS выдержит этот сервис?

**Ответ:** Лимит RPS в данном случае зависит не от кода — он очень лёгкий: просто чтение int под `RLock` и запись строки в ответ. Основные ограничения будут на уровне среды выполнения.

Конкретно, пропускная способность будет зависеть от:
— значения `GOMAXPROCS` — сколько потоков рантайм может использовать параллельно,
— количества ядер CPU — больше ядер = больше параллелизма,
— объёма оперативной памяти и пропускной способности сети

---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Channels 
  - Select
  - Timeout
title: Обёртка с таймаутом для долгой функции
---

## Условие
Есть функция, работающая неопределённо долго (например, сетевой запрос). Её тело нельзя изменять.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func init() {
    rand.Seed(time.Now().UnixNano())
}

func unpredictableFunc() int64 {
    rnd := rand.Int63n(5000)
    time.Sleep(time.Duration(rnd) * time.Millisecond)
    return rnd
}

func predictableFunc() int64 {
    return unpredictableFunc()
}

func main() {
    fmt.Println("started")
    fmt.Println(predictableFunc())
}
```

**Требования:**
1. Написать обёртку с таймаутом (например, 1 секунда)
2. Если функция отработала за это время — вернуть результат
3. Если нет — вернуть ошибку
4. Измерить время выполнения

## Решение

```go
package main

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

func init() {
    rand.Seed(time.Now().UnixNano())
}

func unpredictableFunc() int64 {
    rnd := rand.Int63n(5000)
    time.Sleep(time.Duration(rnd) * time.Millisecond)
    return rnd
}

func predictableFunc(timeout time.Duration) (int64, error) {
    start := time.Now()

    defer func() {
        fmt.Println("Время выполнения:", time.Since(start))
    }()

    resultCh := make(chan int64, 1) // Буферизированный канал!

    go func() {
        val := unpredictableFunc()
        resultCh <- val
        close(resultCh)
    }()

    select {
    case res := <-resultCh:
        return res, nil
    case <-time.After(timeout):
        return 0, errors.New("timeout exceeded")
    }
}

func main() {
    fmt.Println("started")
    res, err := predictableFunc(time.Second)
    if err != nil {
        fmt.Println("Ошибка:", err)
    } else {
        fmt.Println("Результат:", res)
    }
}
```

## Дополнительные вопросы

**А если в select поставить default return — будет работать?**
Нет, `default` выполнится мгновенно, не дожидаясь ни результата, ни таймаута.

**Почему используем буферизированный канал?**
Если таймаут сработал раньше, основная горутина уходит из select. Без буфера горутина с `unpredictableFunc` заблокируется навсегда на `resultCh <- val` (goroutine leak).

**Что будет если убрать буфер из канала?**
Goroutine leak — горутина зависнет на отправке в канал, который никто не читает.

**Что если не закрывать канал?**
В данном случае ничего страшного — канал всё равно будет собран GC после завершения обеих горутин. Но закрывать — хорошая практика.

**А не будет ли в defer time.Since всегда 0?**
Нет, `start` захватывается по значению в момент создания, а `time.Since(start)` вычисляется в момент выполнения defer.

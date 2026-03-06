---
type: task
companies:
  - X5
topic: Go
subtopic:
  - Concurrency
  -  Worker Pool
  -  Semaphore
title: Ограничить количество одновременных горутин
---

## Условие
```go
import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type storage struct {
    result map[int32]string
    mux    sync.Mutex
}

func LongWorker(res *storage, wg *sync.WaitGroup) {
    defer wg.Done()
    time.Sleep(time.Duration(rand.Int31n(10)) * time.Second)

    var key = rand.Int31n(100)
    var value = fmt.Sprintf("some result: %d", rand.Int())

    res.mux.Lock()
    res.result[key] = value
    res.mux.Unlock()
}

func main() {
    s := &storage{result: make(map[int32]string)}
    WorkersCount := rand.Int31n(12) + 1

    var wg sync.WaitGroup
    wg.Add(int(WorkersCount))

    for i := int32(0); i < WorkersCount; i++ {
        go LongWorker(s, &wg)
    }

    wg.Wait()
    fmt.Println(s.result)
}
```

**Вопрос:** Что нужно сделать, если нужно запускать не все горутины сразу, а по две за раз? Как можно ограничить количество одновременно работающих горутин?

## Решение

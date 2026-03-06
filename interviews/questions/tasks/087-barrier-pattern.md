---
type: task
companies:
  - X5
topic: Architecture
subtopic:
  - sync.Cond
  - Mutex
  - Barrier
  - Concurrency Patterns
title: Реализация паттерна барьер — запуск воркеров группами
---

## Условие

Реализовать паттерн синхронизации «барьер». Воркеры запускаются только когда их количество достигло определённого лимита (BSIZE). N воркеров, размер батча BSIZE. Воркеры ждут, пока не наберётся нужное количество, затем запускаются параллельно.

Нужно реализовать методы Add, Wait, Done и TryAwake.

## Пример
```go
var N = 10
var BSIZE = 2

type Barrier struct {
    size    int
    current int
    mu      sync.Mutex
}

func NewBarrier(size int) *Barrier {
    b := new(Barrier)
    return b
}

func (b *Barrier) Add() {
}

func (b *Barrier) TryAwake() {
    awake := false

    if awake {
        println("--\nAWAKE \n")
    }
}

func (b *Barrier) Wait(id int) {
}

func (b *Barrier) Done() {
}

func worker(id int, barrier *Barrier) {
    // тут что-то надо сделать
    fmt.Printf("Worker %d is waiting.\n", id)
    barrier.Wait(id)

    time.Sleep(time.Duration(3*rand.Float32()) * time.Second)
    fmt.Printf("Worker %d is running.\n", id)
    // ...
}

func main() {
    // результат должен быть примерно такой:
    /*
    AWAKE

    Worker 1 is waiting.
    Worker 0 is waiting.
    Worker 1 is running.
    Worker 0 is running.
    --
    AWAKE
    */
}
```

## Решение
```go
```
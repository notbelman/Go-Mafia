---
type: task
companies:
  - MTS
topic: Go
subtopic:
  - Code Review
  - Concurrency
  -  Sampling
title: Do Sender - ревью кода семплирования трейсов
---

## Условие
Описать какую задачу решает данный код. Какие проблемы есть в этом решении? Как бы вы переписали этот код?

```go
var c int

type Trace struct {
    // Some fields here
}

type Sender interface {
    Send(Trace)
}

func Do(sender Sender, tr Trace) {
    c++
    if c == 100 {
        sender.Send(tr)
        c = 0
    }
}
```

## Решение

```go
type Trace struct {
    // Some fields here
}

type Sender interface {
    Send(Trace)
}

type SampledSender struct {
    counter int
    mu      sync.Mutex
}

func New() *SampledSender {
    return &SampledSender{}
}

func (ss *SampledSender) Send(tr Trace) {
    send := false

    ss.mu.Lock()
    ss.counter++
    if ss.counter == 100 {
        ss.counter = 0
        send = true
    }
    ss.mu.Unlock()

    if send {
        sender.Send(tr)
    }
}
```

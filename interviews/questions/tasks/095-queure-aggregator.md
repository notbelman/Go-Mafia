---
type: task
companies:
  - Wildberries
topic: Go
subtopic:
  - Channels
  - Fan-In
  - Aggregation
  - Ticker
  - Context
title: QueueAggregator — агрегация сессий по проектам с флашем по таймеру и по количеству
---

## Условие

Написать агрегатор, который принимает на вход слайс каналов с сессиями (поступают в перемешанном виде по нескольким проектам) и возвращает канал, в который отправляются агрегированные сессии по проектам.

Данные отправляются в канал по одному из двух условий:
1. По таймеру q.cfg.FlushInterval — в канал отправляются все сессии, собранные за этот период
2. По достижении q.cfg.MaxSessionsPerProject — в канал отправляются все собранные сессии по данному проекту

## Пример
```go
import (
    "context"
    "time"
)

type Session struct {
    ProjectID string
    Data      []byte
}

type ProjectSessions struct {
    ProjectID string
    Sessions  []Session
}

type QueueAggregatorCfg struct {
    FlushInterval        time.Duration
    MaxSessionsPerProject int
}

type QueueAggregator struct {
    cfg QueueAggregatorCfg
}

func NewQueueAggregator(cfg QueueAggregatorCfg) *QueueAggregator {
    return &QueueAggregator{cfg: cfg}
}

// Входные данные: слайс каналов chs, где сессии поступают в перемешанном виде
// Выходной результат: канал с агрегированными сессиями по проектам
func (q *QueueAggregator) Run(ctx context.Context, chs []<-chan Session) <-chan ProjectSessions {
}
```

## Решение
```go
```
---
type: task
companies:
  - Астрал-Софт
topic: Go
subtopic:
  - Interfaces
  - Context
  - Concurrency
  - errgroup
  - System Design
title: Улучшить fetch с Querier-интерфейсом — контролируемость и concurrency
---

## Условие

Дан код:

```go
type Result struct {
    // ...
}

type Querier interface {
    Get() Result
}

func fetch(queries []Querier) []Result {
}
```

Как улучшить программу, чтобы она была более контролируема с точки зрения поведения? Что бы добавил, что позволило бы лучше управлять ходом выполнения?

## Решение

### 1. context.Context — отмена и таймауты

```go
type Querier interface {
    Get(ctx context.Context) (Result, error)
}

func fetch(ctx context.Context, queries []Querier) ([]Result, error) {
```

Без контекста функция неуправляема: вызвал и жди бесконечно. Контекст позволяет задать таймаут, дедлайн, отменить все запросы извне.

### 2. Возврат ошибок

Оригинальный `Get() Result` не возвращает ошибку — нет способа понять что пошло не так. Интерфейс должен возвращать `(Result, error)`.

### 3. Параллельное выполнение через errgroup

```go
func fetch(ctx context.Context, queries []Querier) ([]Result, error) {
    results := make([]Result, len(queries))
    g, ctx := errgroup.WithContext(ctx)

    for i, q := range queries {
        g.Go(func() error {
            res, err := q.Get(ctx)
            if err != nil {
                return err
            }
            results[i] = res
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        return nil, err
    }
    return results, nil
}
```

### 4. Ограничение параллелизма

```go
g.SetLimit(10)
```

Без лимита при 10000 queries запустится 10000 горутин — можно положить сервис или удалённый API.

### 5. Дополнительные улучшения

- **Retry с backoff** для transient-ошибок
- **Partial results** вместо fail-all (канал ошибок вместо errgroup)
- **Логирование / трейсинг** через контекст
- **Метрики** (latency, error rate per querier)
- **Graceful degradation** — вернуть то что успели собрать при отмене контекста

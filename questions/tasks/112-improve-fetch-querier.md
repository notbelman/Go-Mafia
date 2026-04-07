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

---

## Решение

```go
// 1. Get() возвращает ошибку — иначе не узнаем что пошло не так
// 2. ctx — отмена и таймауты снаружи
type Querier interface {
    Get(ctx context.Context) (Result, error) // 1, 2
}

// 5. собираем результаты и ошибки отдельно — partial results:
//    одна упавшая query не убивает остальные
type FetchResult struct {
    Results []Result
    Errors  []error
}

// 2. ctx передаём в fetch
// 4. maxConcurrency — лимит горутин (иначе 10k queries = 10k горутин)
func fetch(ctx context.Context, queries []Querier, maxConcurrency int) FetchResult {
    results := make([]Result, len(queries))
    errs := make([]error, len(queries))

    // 3. errgroup — параллельный запуск + ждём всех через Wait()
    g, ctx := errgroup.WithContext(ctx)
    g.SetLimit(maxConcurrency) // 4

    for i, q := range queries {
        g.Go(func() error {
            res, err := q.Get(ctx)
            // 6. или: res, err := fetchWithRetry(ctx, q, 3) — retry + exponential backoff
            if err != nil {
                errs[i] = err // 5. пишем ошибку, не возвращаем — не отменяем остальных
                return nil
            }
            results[i] = res // безопасно без мьютекса: каждая горутина пишет в свой i
            return nil
        })
    }

    g.Wait()
    return FetchResult{Results: results, Errors: errs}
}

ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second) // таймаут на всю операцию
defer cancel()

result := fetch(ctx, queries, 10) // 10 — по размеру connection pool / rate limit API

failed := 0
for _, err := range result.Errors {
    if err != nil {
        failed++
        log.Warn(err)
    }
}
// решаем сами: упало < 10% — используем partial results, иначе возвращаем ошибку наверх

// 6. retry с exponential backoff — временная ошибка не финальный фейл
func fetchWithRetry(ctx context.Context, q Querier, maxRetries int) (Result, error) {
    var lastErr error
    for attempt := 0; attempt <= maxRetries; attempt++ {
        if attempt > 0 {
            backoff := time.Duration(attempt*attempt) * 100 * time.Millisecond // 100ms, 400ms, 900ms
            select {
            case <-time.After(backoff):
            case <-ctx.Done():
                return Result{}, ctx.Err()
            }
        }
        res, err := q.Get(ctx)
        if err == nil {
            return res, nil
        }
        lastErr = err
    }
    return Result{}, fmt.Errorf("after %d retries: %w", maxRetries, lastErr)
}
```

## Альтернатива через семафор (без errgroup)

errgroup внутри использует буферизованный канал как семафор. Можно сделать то же самое вручную через `sync.WaitGroup` + канал:

```go
func fetch(ctx context.Context, queries []Querier, maxConcurrency int) FetchResult {
    results := make([]Result, len(queries))
    errs := make([]error, len(queries))

    sem := make(chan struct{}, maxConcurrency) // семафор: буферизованный канал
    var wg sync.WaitGroup

    for i, q := range queries {
        wg.Add(1)
        sem <- struct{}{} // занимаем слот — блокируемся если уже maxConcurrency горутин
        go func() {
            defer wg.Done()
            defer func() { <-sem }() // освобождаем слот
            res, err := q.Get(ctx)
            if err != nil {
                errs[i] = err
                return
            }
            results[i] = res
        }()
    }

    wg.Wait()
    return FetchResult{Results: results, Errors: errs}
}
```

Отличие от errgroup: больше boilerplate, нет `SetLimit` из коробки — но механика та же.

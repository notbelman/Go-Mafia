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

## Решение (по нарастающей сложности)

### Уровень 0: наивная реализация (что есть сейчас)

```go
func fetch(queries []Querier) []Result {
    results := make([]Result, len(queries))
    for i, q := range queries {
        results[i] = q.Get()
    }
    return results
}
```

Проблемы:

- Последовательное выполнение — если каждый `Get()` ходит в сеть за 100ms, то 100 queries = 10 секунд.
- Нет способа отменить выполнение — вызвал `fetch` и жди сколько угодно.
- `Get()` не возвращает ошибку — если запрос упал, мы об этом не узнаем. Паника? Пустой Result? Непонятно.
- Если один запрос завис навсегда — вся функция зависла навсегда.

---

### Уровень 1: возврат ошибок

**Проблема:** оригинальный `Get() Result` не сообщает об ошибке. Если запрос к БД упал, сеть отвалилась, таймаут — вызывающий код не узнает.

**Решение:** интерфейс должен возвращать `(Result, error)`.

```go
type Querier interface {
    Get() (Result, error)
}

func fetch(queries []Querier) ([]Result, error) {
    results := make([]Result, len(queries))
    for i, q := range queries {
        res, err := q.Get()
        if err != nil {
            return nil, fmt.Errorf("query %d failed: %w", i, err)
        }
        results[i] = res
    }
    return results, nil
}
```

**Что изменилось:**

- Вызывающий код может обработать ошибку: повторить, залогировать, вернуть partial result.
- Сигнатура `fetch` тоже возвращает `error` — ошибка пробрасывается наверх.

**Что всё ещё плохо:** всё последовательно, нет отмены.

---

### Уровень 2: context.Context — отмена и таймауты

**Проблема:** вызвали `fetch` — и нет способа остановить его. Представь: пользователь закрыл страницу, а сервер всё ещё обходит 100 queries. Или деплой — нужен graceful shutdown, а `fetch` не реагирует на сигналы.

**Решение:** передаём `context.Context` и в `fetch`, и в `Get`.

```go
type Querier interface {
    Get(ctx context.Context) (Result, error)
}

func fetch(ctx context.Context, queries []Querier) ([]Result, error) {
    results := make([]Result, len(queries))
    for i, q := range queries {
        res, err := q.Get(ctx)
        if err != nil {
            return nil, fmt.Errorf("query %d failed: %w", i, err)
        }
        results[i] = res
    }
    return results, nil
}
```

**Как это используется вызывающим кодом:**

```go
// Таймаут на всю операцию — 5 секунд
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

results, err := fetch(ctx, queries)
if errors.Is(err, context.DeadlineExceeded) {
    // не успели за 5 секунд
}
```

**Что даёт context:**

- `WithTimeout` / `WithDeadline` — автоматическая отмена по времени.
- `WithCancel` — ручная отмена извне (graceful shutdown, пользователь отменил запрос).
- Каждый `Get(ctx)` внутри себя проверяет `ctx.Done()` и выходит рано, не дожидаясь ответа.

**Что всё ещё плохо:** всё последовательно. 100 queries по 100ms = 10 секунд.

---

### Уровень 3: параллельное выполнение через errgroup

**Проблема:** queries независимы друг от друга, но выполняются одна за другой. Это бессмысленно — можно запустить все параллельно.

**Решение:** `golang.org/x/sync/errgroup` — стандартный инструмент для параллельного выполнения с обработкой ошибок.

```go
import "golang.org/x/sync/errgroup"

func fetch(ctx context.Context, queries []Querier) ([]Result, error) {
    results := make([]Result, len(queries))
    g, ctx := errgroup.WithContext(ctx)

    for i, q := range queries {
        g.Go(func() error {
            res, err := q.Get(ctx)
            if err != nil {
                return fmt.Errorf("query %d failed: %w", i, err)
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

**Разбор по строкам:**

`g, ctx := errgroup.WithContext(ctx)` — создаём группу горутин, привязанную к контексту. Если любая горутина вернёт ошибку — контекст автоматически отменится, и остальные горутины (если проверяют `ctx.Done()`) завершатся рано.

`g.Go(func() error { ... })` — запускает горутину внутри группы. errgroup следит за всеми запущенными горутинами.

`results[i] = res` — безопасно без мьютекса, потому что каждая горутина пишет в свой уникальный индекс `i`. Нет гонки данных.

`g.Wait()` — блокируется до завершения всех горутин. Возвращает первую ошибку (если была).

**Что даёт:**

- 100 queries по 100ms = ~100ms (вместо 10 секунд).
- Если одна упала — контекст отменяется, остальные завершаются рано.
- `Wait()` гарантирует что все горутины завершились перед возвратом — нет утечек.

**Что всё ещё плохо:** если queries 10000 — запустится 10000 горутин одновременно.

---

### Уровень 4: ограничение параллелизма (semaphore)

**Проблема:** без лимита `fetch` с 10000 queries запустит 10000 горутин, каждая из которых полезет в сеть/БД. Это может:

- Исчерпать connection pool к БД.
- Получить rate limit от внешнего API.
- Сожрать память (каждая горутина ~2-8 KB стека).

**Решение:** `errgroup.SetLimit`.

```go
func fetch(ctx context.Context, queries []Querier, maxConcurrency int) ([]Result, error) {
    results := make([]Result, len(queries))
    g, ctx := errgroup.WithContext(ctx)
    g.SetLimit(maxConcurrency) // например 10

    for i, q := range queries {
        g.Go(func() error {
            res, err := q.Get(ctx)
            if err != nil {
                return fmt.Errorf("query %d failed: %w", i, err)
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

**Как работает SetLimit:**

- Внутри errgroup — семафор (буферизованный канал или `sync.Semaphore`).
- `g.Go()` блокируется, если уже запущено `maxConcurrency` горутин.
- Как только одна завершилась — следующая стартует.
- Итого одновременно работают не больше N горутин.

**Как выбрать лимит:**

- Ходим в БД → размер connection pool (обычно 10-50).
- Ходим во внешний API → их rate limit.
- CPU-bound работа → `runtime.NumCPU()`.

---

### Уровень 5: partial results (graceful degradation)

**Проблема:** errgroup при первой ошибке отменяет контекст. `Wait()` возвращает первую ошибку и `nil` results. Но что если из 100 queries упала одна — а 99 результатов всё равно полезны?

**Решение:** собираем результаты и ошибки отдельно, не падаем на первой ошибке.

```go
type FetchResult struct {
    Results []Result
    Errors  []error // ошибки по индексам
}

func fetch(ctx context.Context, queries []Querier, maxConcurrency int) FetchResult {
    results := make([]Result, len(queries))
    errs := make([]error, len(queries))

    g, ctx := errgroup.WithContext(ctx)
    g.SetLimit(maxConcurrency)

    for i, q := range queries {
        g.Go(func() error {
            res, err := q.Get(ctx)
            if err != nil {
                errs[i] = fmt.Errorf("query %d: %w", i, err)
                return nil // не возвращаем ошибку — не отменяем остальных
            }
            results[i] = res
            return nil
        })
    }

    g.Wait()

    return FetchResult{
        Results: results,
        Errors:  errs,
    }
}
```

**Ключевое отличие:** `return nil` вместо `return err` внутри горутины. Ошибка сохраняется в `errs[i]`, но не отменяет контекст и не останавливает другие горутины.

**Вызывающий код решает** что делать:

```go
result := fetch(ctx, queries, 10)
failed := 0
for _, err := range result.Errors {
    if err != nil {
        failed++
        log.Warn(err)
    }
}
// Если упало меньше 10% — используем что есть
// Если упало больше — возвращаем ошибку наверх
```

---

### Уровень 6: retry с backoff

**Проблема:** запрос может упасть из-за временной ошибки (сеть мигнула, 503 от сервиса). Сразу считать это фейлом — расточительно.

**Решение:** оборачиваем `Get` в retry-логику.

```go
func fetchWithRetry(ctx context.Context, q Querier, maxRetries int) (Result, error) {
    var lastErr error
    for attempt := 0; attempt <= maxRetries; attempt++ {
        if attempt > 0 {
            backoff := time.Duration(attempt*attempt) * 100 * time.Millisecond // 100ms, 400ms, 900ms...
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

**Exponential backoff:** `attempt² × 100ms` — 100ms, 400ms, 900ms. Не долбим упавший сервис в цикле, даём ему восстановиться.

**`select` с `ctx.Done()`:** если контекст отменился во время ожидания backoff — выходим сразу, не ждём таймер.

Используем внутри `fetch`:

```go
g.Go(func() error {
    res, err := fetchWithRetry(ctx, q, 3)
    if err != nil {
        errs[i] = err
        return nil
    }
    results[i] = res
    return nil
})
```

---

## Итого: эволюция решения

|Уровень|Что добавили|Какую проблему решили|
|---|---|---|
|0|Наивный цикл|—|
|1|`error` в возврате|Не знаем что пошло не так|
|2|`context.Context`|Нет отмены, нет таймаутов|
|3|`errgroup`|Последовательное выполнение|
|4|`SetLimit`|Неконтролируемый параллелизм|
|5|Partial results|Одна ошибка убивает всё|
|6|Retry + backoff|Временные ошибки = финальный фейл|

На собесе достаточно дойти до уровня 4 и упомянуть 5-6 устно. Это покажет что ты понимаешь production-паттерны Go.
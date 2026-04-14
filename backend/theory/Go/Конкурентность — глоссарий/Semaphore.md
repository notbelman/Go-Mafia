- **Семафор** — счётчик с Acquire (P, уменьшить) и Release (V, увеличить). При 0 — Acquire блокируется ^sem-def
- **Бинарный** (count=1) = мьютекс. **Считающий** (count=N) = ограничитель параллелизма до N горутин ^sem-types
- **Отличие от мьютекса**: у семафора **нет владельца** — Release может вызвать другая горутина ^sem-vs-mutex

---

```go
// Семафор через канал
sem := make(chan struct{}, N)

sem <- struct{}{}  // Acquire — займи слот
// ... работа ...
<-sem              // Release — освободи слот
```
^sem-channel

## Пример: ограничение параллельных HTTP-запросов

```go
sem := make(chan struct{}, 10) // максимум 10 одновременно
for _, url := range urls {
    sem <- struct{}{}
    go func(u string) {
        defer func() { <-sem }()
        http.Get(u)
    }(url)
}
```
^sem-http-example

## golang.org/x/sync/semaphore

**Взвешенный** семафор с контекстом: `sem.Acquire(ctx, weight)`. Горутина может занять несколько слотов. ^sem-weighted

## Связь
- [[Mutex]] — mutex = бинарный семафор с владельцем
- [[Contention]] — семафор снижает contention, ограничивая параллелизм

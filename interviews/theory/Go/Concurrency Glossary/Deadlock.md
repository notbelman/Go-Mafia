- **Deadlock** — два+ потока ждут друг друга **вечно**. A держит лок 1 и ждёт лок 2, B наоборот. **Никто** не продвинется ^dl-def
- **4 условия** (все одновременно → deadlock): mutual exclusion, hold and wait, no preemption, circular wait ^dl-conditions
- Go runtime ловит **только полный** deadlock (все горутины спят). Частичный (2 из 100) — **не обнаружит** ^dl-go-detection

---

```go
// DEADLOCK
go func() { mu1.Lock(); mu2.Lock() }()
go func() { mu2.Lock(); mu1.Lock() }()  // обратный порядок
```
^dl-example

## 4 условия Коффмана

1. **Mutual exclusion** — ресурс эксклюзивный ^dl-cond-mutex
2. **Hold and wait** — держу одно, жду другое ^dl-cond-hold
3. **No preemption** — нельзя отобрать ресурс силой ^dl-cond-preempt
4. **Circular wait** — цикл ожидания A→B→A ^dl-cond-circular

**Убери любое одно** → deadlock невозможен. ^dl-break

## Решения

| Решение | Что убивает |
|:--|:--|
| **Порядок захвата** (всегда mu1 перед mu2) | Circular wait |
| **Таймаут** / TryLock | Hold and wait |
| **Wait-for graph** (граф ожидания) | Обнаружение цикла |
^dl-solutions

## Связь
- [[Livelock]] — все "работают", но не продвигаются
- [[Starvation]] — система работает, но не для всех

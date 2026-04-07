- кэш-линия = 64-128 байт, минимальная единица обмена между ядром и RAM ^fs-cache-line-size
- false sharing: шарды лежат рядом → попадают в одну кэш-линию → ядра инвалидируют кэши друг друга ^fs-definition
- решение: padding между шардами, чтобы каждый занимал отдельную кэш-линию → ускорение в десятки-сотни раз ^fs-solution-summary

---

## Контекст: шардированный атомарный счётчик

Пишем библиотеку метрик (Prometheus, VictoriaMetrics). Счётчик инкрементируется часто (каждый запрос), читается редко (скрапинг). Шардируем по ядрам:

```go
type ShardedCounter struct {
    shards [16]atomic.Int64  // по шарду на ядро
}
func (c *ShardedCounter) Inc(goroutineID int) {
    c.shards[goroutineID % 16].Add(1)
}
func (c *ShardedCounter) Get() int64 {
    var sum int64
    for i := range c.shards { sum += c.shards[i].Load() }
    return sum
}
```

Идея здравая — ядра не спотыкаются на одном атомике. Но ускорение минимальное. Почему? ^fs-sharded-context

## Проблема

atomic.Int64 = 8 байт. Кэш-линия = 64 байт. В одну кэш-линию влезает 8 шардов: ^fs-problem-math

```
Кэш-линия (64 байта):
┌────────┬────────┬────────┬────────┬────────┬────────┬────────┬────────┐
│shard[0]│shard[1]│shard[2]│shard[3]│shard[4]│shard[5]│shard[6]│shard[7]│
└────────┴────────┴────────┴────────┴────────┴────────┴────────┴────────┘
```

Ядро 0 инкрементирует shard[0] → **вся кэш-линия** инвалидируется у всех ядер → ядро 1 перечитывает shard[1] из RAM, хотя shard[1] не менялся. Это **false sharing** — ложное разделение. ^fs-invalidation-mechanism

## Решение: padding

```go
type PaddedCounter struct {
    value atomic.Int64
    _     [56]byte  // 64 - 8 = 56 байт padding
}

type ShardedCounter struct {
    shards [16]PaddedCounter  // каждый шард в своей кэш-линии
}
```

Теперь каждый шард занимает ровно 64 байта = одну кэш-линию. Ядра больше не мешают друг другу. ^fs-padding-solution

Padding 128 байт даёт ещё больший прирост — кэш-линия может быть 128 байт на некоторых архитектурах. ^fs-padding-128

## Бенчмарк (из лекции)

| Подход | Результат |
|:-------|:----------|
| Mutex | baseline |
| Atomic (один) | ~2x быстрее |
| Шарды без padding | чуть быстрее atomic |
| Шарды + padding 64 | **в десятки раз быстрее** |
| Шарды + padding 128 | ещё быстрее (кэш-линия может быть 128) |

^fs-benchmark

## True sharing vs False sharing

**True sharing:** все ядра работают с одним атомиком — инвалидация неизбежна, это цена синхронизации. ^fs-true-sharing

**False sharing:** ядра работают с разными данными, но данные лежат рядом в памяти — инвалидация ложная, решается padding. ^fs-false-sharing-def

## Связь
- [[Atomic vs Mutex]] — атомик ~2x быстрее мьютекса, но шардирование + padding ещё быстрее
- [[sync - atomic]] — атомарные операции
- [[WORK-BASE/interviews/theory/Go/sync/sync.Pool]] — в Pool тоже per-P + padding (poolLocal)

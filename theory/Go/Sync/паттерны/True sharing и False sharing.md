- True sharing — потоки **реально** работают с одними данными → contention неизбежен, это цена дизайна
- False sharing — потоки работают с **разными** данными, но они в одной кэш-линии → ложный contention, чинится padding
- Оба про **кэш-когерентность** (MESI протокол): запись в кэш-линию инвалидирует её на других ядрах

---

## Как работает кэш (коротко)

CPU не читает по одному байту — читает **кэш-линиями** (обычно 64 байта). Если ядро 0 пишет в кэш-линию, протокол когерентности (MESI) **инвалидирует** эту линию на всех других ядрах. Следующий read с другого ядра — cache miss, идём в RAM. ^cache-line-size

```
Ядро 0: [====== 64 bytes ======]  ← пишет
Ядро 1: [====== 64 bytes ======]  ← ИНВАЛИДИРОВАНО, cache miss
```

^mesi-invalidation

## True sharing

Потоки **действительно** читают/пишут одну переменную:

```go
var counter atomic.Int64

// 4 горутины инкрементят один счётчик
for i := 0; i < 4; i++ {
    go func() {
        for j := 0; j < 1_000_000; j++ {
            counter.Add(1)  // все пишут в один адрес
        }
    }()
}
```

Каждый `Add(1)` инвалидирует кэш-линию `counter` на остальных ядрах. Это **неизбежно** — данные реально разделяемые. ^true-sharing-def

**Как смягчить:**
- Sharding: каждый поток считает в свой счётчик, в конце суммируем
- Батчинг: накапливаем локально, пишем редко ^true-sharing-mitigation

## False sharing

Потоки работают с **разными** переменными, но те лежат в одной кэш-линии:

```go
type Counters struct {
    a int64  // горутина 0 пишет сюда
    b int64  // горутина 1 пишет сюда
}
// a и b рядом в памяти → одна кэш-линия!
```

```
Кэш-линия: [.... a .... b ....]
Ядро 0 пишет a → инвалидирует линию на ядре 1
Ядро 1 пишет b → инвалидирует линию на ядре 0
// Пинг-понг! Хотя данные РАЗНЫЕ
```

^false-sharing-def

**Результат**: кэш-линия летает между ядрами, performance деградирует в 10-50x. ^false-sharing-perf-impact

## Как чинить false sharing — padding

```go
type Counters struct {
    a int64
    _ [56]byte  // padding до 64 байт (размер кэш-линии)
    b int64
    _ [56]byte
}
// теперь a и b в РАЗНЫХ кэш-линиях
```

Или `cacheline` alignment:

```go
type PaddedInt64 struct {
    val int64
    _   [cache.CacheLinePad]byte  // Go: golang.org/x/sys/cpu
}
```

^false-sharing-padding

## Пример в Go runtime

`runtime.p` (структура P) содержит `runq` — каждый P имеет свою LRQ. Если бы LRQ разных P лежали в одной кэш-линии — false sharing убил бы производительность. Поэтому P выровнены по кэш-линиям. ^go-runtime-p-alignment

## Как диагностировать

- **perf stat** (Linux): `L1-dcache-load-misses` — аномально высокий при false sharing ^diag-perf-stat
- **perf c2c** (Linux): показывает именно false sharing — кэш-линии с contention ^diag-perf-c2c
- Бенчмарк: сравнить с padding и без — если разница >2x, скорее всего false sharing ^diag-benchmark

## Сравнение

```
                True sharing          False sharing
Данные         одни и те же           разные
Contention     неизбежен              ложный (артефакт layout)
Решение        sharding, батчинг     padding, выравнивание
Пример         atomic counter        struct { a, b int64 }
```

^sharing-comparison

## Связь
- [[CAS паттерны]] — CAS вызывает cache line bouncing (true sharing)
- [[Spin lock]] — все потоки молотят один `locked` — true sharing
- [[Ticket lock]] — все читают `owner` — true sharing
- [[Внутреннее устройство очередей]] — P выровнены чтобы избежать false sharing
- [[Шардированная мапа]]

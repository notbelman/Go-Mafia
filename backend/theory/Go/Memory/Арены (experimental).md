- region-based memory management = **линейный аллокатор**: наплодил объектов → Free() всё махом
- чанки по 8MB, связный список. API: `arena.New[T]()`, `MakeSlice`, `Clone`, `Free`
- **не синхронизированы** (одна горутина), reference types — вручную, map нельзя
- use-after-free = **undefined behavior** (как в C/C++), нужны sanitizers

---
[[memory Flashcards - arenas]]
## Идея

Батчевая обработка: приходит большой батч данных, обрабатываем, освобождаем всё махом. Не нагружаем GC поштучной очисткой. ^arena-idea

## API (experimental: нужен тег `goexperiment.arenas`)

Для использования арен нужен build tag `goexperiment.arenas`. Арена выделяет чанки по 8MB, внутри — связный список чанков. ^arena-internals

```go
a := arena.NewArena()
defer a.Free()  // освобождает ВСЮ память арены

num := arena.New[int64](&a)        // аллокация int64 в арене
*num = 42

data := arena.New[MyStruct](&a)    // структура в арене
slice := arena.MakeSlice[int](&a, 0, 100)  // срез в арене
clone := arena.Clone(someObj)       // клон → хип (не арена)
```
^arena-api

`arena.Clone` копирует объект в **хип**, а не в арену. ^arena-clone

## Ограничения

**Reference types не аллоцируются автоматически:**

```go
type Data struct {
    Operations []Op  // underlying array НЕ попадёт в арену!
}

// Вручную:
ops := arena.MakeSlice[Op](&a, 0, 10)  // массив в арене
data := arena.New[Data](&a)
data.Operations = ops                    // привязываем
```
^arena-ref-types

**Другие ограничения:**

`map` нельзя в арену. ^arena-no-map

Строки — через `MakeSlice[byte]` + unsafe-каст. `append` за пределы cap → новый массив уедет в хип, не в арену. ^arena-strings-append

Арена не синхронизирована — только из одной горутины (sync.Pool — из любой). ^arena-no-sync

## Use-after-free = UB

```go
a := arena.NewArena()
data := arena.New[MyStruct](&a)
a.Free()

data.Field = 42  // 💥 undefined behavior!
// Нет паники. Непредсказуемый результат: мусор, чужие данные, тишина.
```
^arena-uaf

Как в C/C++: используете память после освобождения → неопределённое поведение. Для отлова: memory sanitizer (`-msan`), address sanitizer (`-asan`), работают на Linux. ^arena-sanitizers

## sync.Pool vs Арены

sync.Pool: переиспользование однотипных объектов, синхронизирован, GC может забрать. ^arena-vs-pool-syncpool

Арены: батчевое освобождение разнотипных объектов, не синхронизированы, ручное управление, UB при ошибках. ^arena-vs-pool-arena

## Связь
- [[sync.Pool]] — альтернатива: пул переиспользуемых объектов
- [[Алгоритмы аллокации (база)]] — арена = линейный аллокатор
- [[Практические приёмы уменьшения аллокаций]] — другие способы

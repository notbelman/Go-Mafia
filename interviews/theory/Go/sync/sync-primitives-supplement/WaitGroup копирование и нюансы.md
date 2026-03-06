- передача WaitGroup по значению = Done над копией → оригинал не изменится → deadlock ^wg-copy-bug
- правило: объекты из пакета sync **не копировать**, передавать по указателю ^wg-no-copy-rule
- Add(n) один раз вместо n×Add(1): атомарный инкремент ~15x медленнее обычного ^wg-add-optimization

---

## Копирование = баг

```go
func worker(wg sync.WaitGroup) {  // ← копия!
    defer wg.Done()                // Done над копией
    // работа
}

func main() {
    var wg sync.WaitGroup
    wg.Add(1)
    go worker(wg)
    wg.Wait()  // deadlock: оригинальный счётчик не декрементировался
}
```

WaitGroup — структура со счётчиком и семафором. Копия = отдельный объект. Done над копией не затрагивает оригинал. ^wg-copy-internals

**Решение:** передавать по указателю:

```go
func worker(wg *sync.WaitGroup) {
    defer wg.Done()
    // работа
}
```

**Правило:** объекты из пакета sync (Mutex, WaitGroup, Cond, Once, Pool) не должны копироваться после первого использования. Это может приводить к непонятным багам, которые стреляют в рандомные моменты. ^wg-sync-no-copy-list

## Оптимизация: Add(n) вместо n × Add(1)

```go
// ПЛОХО: n атомарных инкрементов
for i := 0; i < n; i++ {
    wg.Add(1)
    go worker(&wg)
}

// ЛУЧШЕ: один атомарный инкремент
wg.Add(n)
for i := 0; i < n; i++ {
    go worker(&wg)
}
```

Атомарный инкремент ~15x медленнее обычного. Одна операция Add(n) вместо n операций Add(1) — меньше overhead. Плюс сразу видно, сколько горутин ожидаем. ^wg-add-n-reason

## Wait() при счётчике = 0

```go
var wg sync.WaitGroup
wg.Wait()  // не блокирует, просто продолжает
```

Никакой магии: если счётчик 0, Wait() сразу возвращается. ^wg-wait-zero

## Add(-10) при малом счётчике = паника

```go
var wg sync.WaitGroup
wg.Add(5)
wg.Add(-10)  // panic: sync: negative WaitGroup counter
```

^wg-negative-panic

## Escape analysis — обычные правила

WaitGroup — обычная структура. Если указатель на неё утекает из функции → аллокация в heap. Никакой магии для sync-примитивов, те же правила escape analysis. ^wg-escape-analysis

## Связь
- [[sync.WaitGroup]] — внутреннее устройство
- [[sync.Wg пример]] — пошаговый пример работы
- [[interviews/theory/Go/sync/sync-primitives-supplement/Deadlock]] — копирование WaitGroup → deadlock

- Happens-before — если A happens-before B, то эффекты A гарантированно видны в B ^hb-definition
- В Go: atomic.Store создаёт happens-before для последующего atomic.Load того же адреса ^hb-atomic-store-load
- Это позволяет синхронизировать НЕатомарные данные через атомарный флаг — без мьютекса ^hb-sync-nonatomic

---
[[memory Flashcards - happens_before]]
## Проблема без happens-before

```go
var msg string
var flag bool

go func() {
    msg = "hello"   // (A)
    flag = true     // (B) — может выполниться ДО (A)
}()

for !flag { runtime.Gosched() }
fmt.Println(msg)    // может быть "" из-за reordering
```

Data race на msg и flag. Компилятор/процессор не знает про другой поток → реордерит. ^hb-problem-example

## Решение — atomic

```go
var msg string
var flag atomic.Bool

go func() {
    msg = "hello"        // (A)
    flag.Store(true)     // (B) — ПОЛНЫЙ БАРЬЕР
}()

for !flag.Load() {       // (C) — ПОЛНЫЙ БАРЬЕР
    runtime.Gosched()
}
fmt.Println(msg)          // (D) — гарантированно "hello"
```

**Цепочка happens-before:**

```
(A) msg = "hello"
      ↓ happens-before (барьер не даёт перепрыгнуть)
(B) flag.Store(true)
      ↓ happens-before (atomic Store → Load)
(C) flag.Load() == true
      ↓ happens-before (барьер не даёт перепрыгнуть)
(D) fmt.Println(msg)
```

^9ba76b

Итого: A happens-before D. Запись msg видна при чтении msg. **Без мьютекса.** ^hb-solution-chain

## Почему race detector не ругается

Race detector понимает happens-before семантику atomic. Если доступ к msg упорядочен через atomic flag — это не data race, а корректная синхронизация через ordering. ^hb-race-detector

## Go atomic = полный барьер

В Go нет выбора типа барьера (в отличие от C++ `memory_order_relaxed`, `_acquire`, `_release`, `_seq_cst`). Все atomic операции — **полный барьер** (sequential consistency). ^hb-go-full-barrier

Это проще (Go = простой язык), но потенциально медленнее чем acquire/release в C++. ^hb-go-vs-cpp

## Другие источники happens-before в Go

- `ch <- v` happens-before `<-ch` (для того же канала) ^hb-channel
- `mu.Lock()` happens-before следующий `mu.Lock()` (для того же мьютекса) ^hb-mutex
- `go f()` happens-before начало выполнения `f()` ^hb-goroutine-start
- `sync.Once.Do(f)` happens-before возврат из любого `Do` ^hb-once

Полный список — Go Memory Model: https://go.dev/ref/mem

## Когда использовать

Синхронизация через atomic ordering — **продвинутая техника**. Используй когда: ^hb-when-use

- Мьютекс слишком тяжёл (hot path) ^hb-use-hot-path
- Простой паттерн "флаг готовности" ^hb-use-flag
- Понимаешь happens-before ^hb-use-understand

В остальных случаях — мьютекс безопаснее и понятнее. ^hb-default-mutex

## Связь
- [[Reordering инструкций]] — проблема, которую решает happens-before
- [[Барьеры памяти ]] — механизм под капотом
- [[Примитивный мьютекс]] — мьютекс тоже создаёт happens-before (через acquire/release)
- [[CAS паттерны]] — CAS = atomic = полный барьер в Go
- [[RCU]]

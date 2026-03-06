- обычные невыровненные операции — медленнее, но работают. Atomic — **panic**
- CPU должен выполнить atomic за одну инструкцию → данные в двух блоках невозможно
- решения: int64 первым полем, либо `atomic.Int64` (сам следит)

---

[[memory Flashcards - atomic_alignment]]

Обычные операции с невыровненным int64 просто медленнее — два чтения вместо одного, но работают. ^align-regular-ops

Atomic — особый случай. CPU должен выполнить операцию за одну инструкцию, иначе это не atomic. Одна инструкция не может работать с данными в двух блоках памяти — поэтому panic. ^align-atomic-panic-reason

**Проблема на 32-bit:**
```go
type Bad struct {
    flag bool
    counter int64  // offset 4, не кратен 8
}

atomic.AddInt64(&b.counter, 1)  // panic на 32-bit
```
^align-32bit-problem

На 64-bit Go выравнивает int64 по 8 байт. На 32-bit — только по 4. ^align-platform-diff

**Решения:**
```go
// 1. int64 первым полем
type Good struct {
    counter int64  // offset 0
    flag bool
}

// 2. atomic.Int64 (сам следит за выравниванием)
var counter atomic.Int64
```
^align-solutions

## Связь
- [[Выравнивание (alignment)]] — почему выравнивание важно для CPU

- Если A **happens-before** B — эффекты A **гарантированно видны** в B. Нет HB = потенциальный data race ^hb-def
- Одного числа достаточно: не нужна одновременность, нужна **видимость** записей ^hb-visibility
- HB — **транзитивно**: A→B и B→C значит A→C ^hb-transitive

---

## Гарантии в Go

| Операция A | Happens-before операция B |
|:--|:--|
| `ch <- x` | `<-ch` (send → receive completes) |
| `close(ch)` | `<-ch` возвращает zero value |
| `mu.Unlock()` | следующий `mu.Lock()` |
| `wg.Done()` | `wg.Wait()` возвращает |
| `once.Do(f)` | любой `once.Do()` возвращает |
| `atomic.Store` | `atomic.Load` того же адреса |
^hb-guarantees

## Пример: цепочка

```go
var a string
var done = make(chan bool)

go func() {
    a = "hello"   // 1
    done <- true  // 2
}()
<-done            // 3
print(a)          // 4 — гарантированно "hello"
```
^hb-example

`1→2` (sequenced-before) + `2→3` (channel send→receive) = `1→4` (happens-before через транзитивность) ^hb-chain

## Чего HB НЕ гарантирует

**Не гарантирует порядок выполнения во времени** — гарантирует только **видимость**. Операция может физически выполниться раньше, но без HB результат не обязан быть виден. ^hb-not-ordering

## Связь
- [[Data Race]] — нет HB = data race = UB
- [[Барьеры памяти]] — HB реализуется через memory barriers на уровне CPU
- [[Happens-before через atomic]] — atomic.Store/Load создаёт HB

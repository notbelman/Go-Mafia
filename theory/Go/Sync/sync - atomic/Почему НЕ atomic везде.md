### 1. Защищает только одну переменную

```go
// СЛОМАНО
s.count.Add(1)
// <-- здесь другой поток читает: count +1, sum ещё старый
s.sum.Add(val)

// С mutex всё внутри Lock/Unlock -- одна транзакция
mu.Lock()
s.count++
s.sum += val
mu.Unlock()
```

^not-atomic-multi-var

Atomic не даёт гарантий консистентности между двумя разными переменными — между двумя `Add` есть окно, в которое может зайти другой поток. ^not-atomic-multi-var-why

### 2. Read-then-write не атомарен

```go
// СЛОМАНО: между Load и Add кто-то изменит
if counter.Load() < 100 {
    counter.Add(1)  // уже может быть >= 100
}
```

^not-atomic-read-write

Надо CAS loop — но это сложно и error-prone. ^not-atomic-cas-loop-needed

### 3. Lock-free код — это ад

- ABA problem ^not-atomic-aba
- Memory ordering на ARM/PowerPC ^not-atomic-arm-ordering
- 10^24 возможных interleavings при 4 потоках и 10 операциях ^not-atomic-interleavings
- Unit тесты покрывают ~1000 из них ^not-atomic-tests-coverage
- Баги невоспроизводимы ^not-atomic-bugs-unreproducible

## Связь
- [[sync - atomic]] — API atomic
- [[Atomic vs Mutex]] — сравнение
- [[CAS и CAS loop]] — сложность lock-free

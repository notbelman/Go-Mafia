```go
func (rw *RWMutex) RLock() {
    // readerCount += 1, возвращает новое значение
    // "я читатель, захожу"
    //
    // >= 0 → писателя нет → выходим (fast path)
    // < 0  → писатель есть → спим на семафоре
    if rw.readerCount.Add(1) < 0 {
        // усыпляет горутину
        // проснёмся когда писатель вызовет Unlock()
        runtime_SemacquireRWMutexR(&rw.readerSem, false, 0)
    }
}
```

## Как RLock() определяет наличие писателя

`RLock()` атомарно инкрементирует `readerCount` и проверяет знак результата. Если `readerCount < 0` — значит `Lock()` уже вычел `rwmutexMaxReaders`, писатель ждёт или работает. Читатель засыпает на `readerSem`. ^rlock-detect-writer

## Fast path и slow path

**Fast path** (нет писателя): `readerCount.Add(1) >= 0` → выходим, без блокировки. ^rlock-fast-path

**Slow path** (есть писатель): `readerCount.Add(1) < 0` → `runtime_SemacquireRWMutexR(&rw.readerSem)` → горутина засыпает. Проснётся когда писатель вызовет `Unlock()` и сделает `Semrelease(readerSem)`. ^rlock-slow-path

## Связь
- [[sync.RWMutex]] — структура: readerCount
- [[RUnlock()]] — парная операция
- [[Lock() (RWMutex)]] — как writer блокирует новых readers

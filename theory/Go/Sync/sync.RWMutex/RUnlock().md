```go
func (rw *RWMutex) RUnlock() {
    // readerCount -= 1, возвращает новое значение
    // "я читатель, ухожу"
    r := rw.readerCount.Add(-1)
    
    // r >= 0 → писателя нет → выходим (fast path)
    // r < 0  → писатель ждёт → slow path
    if r < 0 {
        rw.rUnlockSlow(r)
    }
}

func (rw *RWMutex) rUnlockSlow(r int32) {
    // r+1 — значение ДО нашего Add(-1)
    // r+1 == 0 → было 0 читателей → RUnlock без RLock
    // r+1 == -rwmutexMaxReaders → только писатель, читателей не было
    if r+1 == 0 || r+1 == -rwmutexMaxReaders {
        fatal("sync: RUnlock of unlocked RWMutex")
    }
    
    // readerWait -= 1
    // readerWait — сколько "старых" читателей писатель ещё ждёт
    // == 0 → мы последний, будим писателя
    if rw.readerWait.Add(-1) == 0 {
        runtime_Semrelease(&rw.writerSem, false, 1)
    }
}
```

## Fast path и slow path

**Fast path** (писателя нет): `readerCount.Add(-1) >= 0` → выходим, без лишней работы. ^runlock-fast-path

**Slow path** (писатель ждёт): `readerCount.Add(-1) < 0` → вызываем `rUnlockSlow`. Декрементируем `readerWait`. Если стало 0 — мы последний "старый" читатель, будим писателя через `Semrelease(writerSem)`. ^runlock-slow-path

## Как RUnlock() определяет "я последний"?

`readerWait.Add(-1) == 0` — атомарный декремент вернул ноль, значит больше нет "старых" читателей которых ждёт писатель. Именно этот читатель отвечает за пробуждение писателя. ^runlock-last-reader

## Два условия fatal в rUnlockSlow

`r+1` — значение `readerCount` **до** нашего `Add(-1)`. ^runlock-fatal-logic

- `r+1 == 0`: до нашего вызова было 0 читателей → `RUnlock` без парного `RLock`
- `r+1 == -rwmutexMaxReaders`: только писатель держит лок, читателей не было → `RUnlock` без `RLock`

^runlock-fatal-cases

## Связь
- [[RLock()]] — парная операция
- [[sync.RWMutex]] — readerCount и readerWait
- [[Unlock() (RWMutex)]] — последний reader будит writer

```go
func (rw *RWMutex) Unlock() {
    // readerCount += rwmutexMaxReaders
    // возвращаем readerCount в положительное значение
    // теперь новые RLock() не будут блокироваться
    //
    // r — сколько читателей ждут на readerSem
    // (они вызвали RLock() пока мы держали лок)
    r := rw.readerCount.Add(rwmutexMaxReaders)
    
    // >= rwmutexMaxReaders → Lock() не вызывался
    // (readerCount был положительным → стал >= rwmutexMaxReaders)
    if r >= rwmutexMaxReaders {
        fatal("sync: Unlock of unlocked RWMutex")
    }
    
    // будим всех ожидающих читателей
    // каждый Semrelease будит одну горутину на readerSem
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem, false, 0)
    }
    
    // отпускаем внутренний мьютекс
    // теперь следующий писатель может войти
    rw.w.Unlock()
}
```

## Как Unlock() узнаёт сколько читателей ждёт

После `readerCount.Add(rwmutexMaxReaders)` результат `r` — это количество читателей которые заблокировались на `readerSem` пока писатель держал лок. Они добавляли `+1` к отрицательному счётчику, поэтому их количество сохранилось в низких битах. ^unlock-r-meaning

## Как Unlock() детектирует некорректный вызов

Если `Lock()` не вызывался, `readerCount` был неотрицательным. После `Add(rwmutexMaxReaders)` результат будет `>= rwmutexMaxReaders` → `fatal`. ^unlock-fatal

## Порядок: сначала читатели, потом писатель

```
Unlock():
  1. readerCount += rwmutexMaxReaders  // разрешаем новые RLock()
  2. будим ждущих читателей            // r раз Semrelease(readerSem)
  3. w.Unlock()                        // впускаем следующего писателя
```
^unlock-order

Это предотвращает reader starvation — после писателя читатели получают шанс войти раньше следующего писателя. ^unlock-reader-starvation

## Связь
- [[WORK-BASE/interviews/theory/Go/sync/sync.RWMutex/Lock()]] — парная операция
- [[Два режима]] — unlockSlow: Normal vs Starvation ветки
- [[WORK-BASE/interviews/theory/Go/str/структура]] — state и sema

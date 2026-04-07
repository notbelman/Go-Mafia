```go
type RWMutex struct {
    w           Mutex        // мьютекс между писателями
    writerSem   uint32       // семафор: писатель спит тут
    readerSem   uint32       // семафор: читатели спят тут
    readerCount atomic.Int32 // счётчик читателей + флаг "есть писатель"
    readerWait  atomic.Int32 // сколько "старых" читателей ждёт писатель
}
```

### Когда меняются поля

`readerCount` — активные + ждущие читатели. Отрицательный = писатель ждёт/работает. ^rwmx-readercount-def

```
RLock()   → +1
RUnlock() → -1
Lock()    → -1млрд (rwmutexMaxReaders ≈ 1_000_000_000)
Unlock()  → +1млрд
```
^rwmx-readercount-ops

`readerWait` — сколько "старых" читателей (тех кто был внутри до прихода писателя) писатель ещё ждёт. ^rwmx-readerwait-def

```
Lock()    → +r (где r = текущие активные читатели)
RUnlock() → -1 (только если readerCount < 0, т.е. писатель ждёт)
```
^rwmx-readerwait-ops

### Что такое семафор?

Семафор — это **счётчик с очередью ожидания**. ^rwmx-semaphore-def

```
runtime_Semacquire(&sem)  // "хочу войти"
  → если sem > 0: sem-- и идём дальше
  → если sem == 0: засыпаем в очереди

runtime_Semrelease(&sem)  // "выхожу, буди следующего"
  → sem++
  → будим одного из очереди (если есть)
```
^rwmx-semaphore-api

В RWMutex семафоры используются как **точки ожидания**: ^rwmx-semaphore-roles

- `writerSem` — писатель спит пока читатели уходят
- `readerSem` — читатели спят пока писатель работает

### Пример

```
Читатель хочет войти, но писатель держит лок:
  RLock() → readerCount < 0 → runtime_Semacquire(&readerSem) → спим

Писатель заканчивает:
  Unlock() → runtime_Semrelease(&readerSem) → будим читателя
```
^rwmx-semaphore-example

По сути семафор = "зал ожидания" с механизмом "следующий!" ^rwmx-semaphore-analogy

## Связь
- [[RLock()]] — захват на чтение
- [[Lock() (RWMutex)]] — захват на запись
- [[sync.Mutex vs sync.RWMutex]] — когда что использовать

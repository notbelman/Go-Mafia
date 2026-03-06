```go
func (rw *RWMutex) Lock() {
    // захватываем внутренний мьютекс
    // другие писатели будут ждать здесь
    rw.w.Lock()
    
    // readerCount -= rwmutexMaxReaders (~1 млрд)
    // делаем readerCount отрицательным → новые RLock() заблокируются
    // + rwmutexMaxReaders в конце → получаем реальное число читателей
    r := rw.readerCount.Add(-rwmutexMaxReaders) + rwmutexMaxReaders
    
    // r != 0        → есть активные читатели
    // readerWait.Add(r) != 0 → они ещё не все ушли
    //
    // readerWait — сколько "старых" читателей мы ждём
    // каждый RUnlock() делает readerWait -= 1
    // последний читатель разбудит нас
    if r != 0 && rw.readerWait.Add(r) != 0 {
        // спим пока последний читатель не вызовет RUnlock()
        runtime_SemacquireRWMutex(&rw.writerSem, false, 0)
    }
}
```

## Механизм блокировки новых читателей

`Lock()` вычитает `rwmutexMaxReaders` (~1 млрд) из `readerCount`, делая его отрицательным. После этого любой `RLock()` увидит отрицательный `readerCount` и заблокируется на `readerSem`. Таким образом новые читатели не могут войти пока писатель ждёт. ^lock-block-readers

## Как Lock() узнаёт реальное число активных читателей

```go
r := rw.readerCount.Add(-rwmutexMaxReaders) + rwmutexMaxReaders
```

После вычитания `rwmutexMaxReaders` значение становится `activeReaders - rwmutexMaxReaders`. Прибавив `rwmutexMaxReaders` обратно, получаем `r = activeReaders`. ^lock-r-formula

## Почему два условия в if?

```go
if r != 0 && rw.readerWait.Add(r) != 0
```

Между `readerCount.Add()` и `readerWait.Add()` читатели могли уйти: ^lock-two-conditions

```
r = 3 (было 3 читателя)
// пока мы тут, все 3 вызвали RUnlock()
// каждый сделал readerWait.Add(-1)
// readerWait стал -3
readerWait.Add(3) → -3 + 3 = 0 → не спим!
```

Первое условие (`r != 0`) — оптимизация: если читателей не было вообще, не трогаем `readerWait`. Второе (`readerWait.Add(r) != 0`) — защита от race когда читатели успели уйти за время между двумя атомарными операциями. ^lock-two-conditions-detail

## Связь
- [[Структура]] — state и sema
- [[Unlock()]] — парная операция
- [[Два режима]] — Normal vs Starvation влияют на алгоритм
- [[state]] — CAS над полем state

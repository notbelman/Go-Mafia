#flashcards/rwmutex/unlock

После `readerCount.Add(rwmutexMaxReaders)` в `Unlock()` результат `r = 4`. Что это значит?
?
![[Unlock()#^unlock-r-meaning]]

Как `Unlock()` детектирует вызов без парного `Lock()`?
?
![[Unlock()#^unlock-fatal]]

Опиши порядок трёх действий в `Unlock()`. Почему именно такой порядок?
?
![[Unlock()#^unlock-order]]
![[Unlock()#^unlock-reader-starvation]]

Почему `Unlock()` будит читателей до вызова `w.Unlock()`? Что было бы если поменять порядок?
?
![[Unlock()#^unlock-order]]
![[Unlock()#^unlock-reader-starvation]]

Что выведет этот код (какой порядок вывода)?
```go
var rw sync.RWMutex
rw.Lock()
go func() { rw.RLock(); fmt.Println("R1"); rw.RUnlock() }()
go func() { rw.RLock(); fmt.Println("R2"); rw.RUnlock() }()
go func() { rw.Lock(); fmt.Println("W2"); rw.Unlock() }()
time.Sleep(50 * time.Millisecond)
rw.Unlock()
time.Sleep(50 * time.Millisecond)
```
?
R1 и R2 (в любом порядке) напечатаются до W2. `Unlock()` сначала будит всех ждущих читателей (`Semrelease(readerSem)` × 2), затем вызывает `w.Unlock()` — только после этого W2 может захватить лок.
![[Unlock()#^unlock-order]]
![[Unlock()#^unlock-reader-starvation]]

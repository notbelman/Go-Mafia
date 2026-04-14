#flashcards/rwmutex/rlock

Как `RLock()` определяет есть ли активный писатель? Почему достаточно проверить знак?
?
![[RLock()#^rlock-detect-writer]]

Опиши fast path и slow path в `RLock()`. Что происходит в каждом случае на уровне runtime?
?
![[RLock()#^rlock-fast-path]]
![[RLock()#^rlock-slow-path]]

Читатель вызвал `RLock()` и заблокировался. Кто его разбудит и когда?
?
![[RLock()#^rlock-slow-path]]

Что выведет этот код?
```go
var rw sync.RWMutex
rw.Lock()
done := make(chan struct{})
go func() {
    rw.RLock()
    fmt.Println("reader in")
    rw.RUnlock()
    close(done)
}()
time.Sleep(10 * time.Millisecond)
fmt.Println("unlocking writer")
rw.Unlock()
<-done
```
?
Сначала "unlocking writer", потом "reader in". Горутина-читатель блокируется на `readerSem` т.к. `readerCount < 0`. После `Unlock()` писатель делает `Semrelease(readerSem)` — читатель просыпается.
![[RLock()#^rlock-slow-path]]

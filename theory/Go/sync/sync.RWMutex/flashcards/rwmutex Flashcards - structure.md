#flashcards/rwmutex/structure

Что хранит поле `readerCount` в `sync.RWMutex` и как по нему определяется наличие писателя?
?
![[sync.RWMutex#^rwmx-readercount-def]]

Перечисли все операции которые изменяют `readerCount` и на сколько каждая из них меняет значение.
?
![[sync.RWMutex#^rwmx-readercount-ops]]

Что хранит поле `readerWait` в `sync.RWMutex`? Чем "старые" читатели отличаются от обычных?
?
![[sync.RWMutex#^rwmx-readerwait-def]]

Какие операции изменяют `readerWait` и при каком условии `RUnlock()` его трогает?
?
![[sync.RWMutex#^rwmx-readerwait-ops]]

Что такое семафор в контексте Go runtime? Опиши семантику `Semacquire` и `Semrelease`.
?
![[sync.RWMutex#^rwmx-semaphore-def]]
![[sync.RWMutex#^rwmx-semaphore-api]]

Для чего используются `writerSem` и `readerSem` в `RWMutex`? Кто на каком спит?
?
![[sync.RWMutex#^rwmx-semaphore-roles]]

Что выведет этот код (опиши последовательность событий)?
```go
var rw sync.RWMutex
rw.RLock()
rw.RLock()
go func() {
    rw.Lock() // пытается взять write lock
    fmt.Println("writer in")
    rw.Unlock()
}()
time.Sleep(10 * time.Millisecond)
rw.RUnlock()
rw.RUnlock()
```
?
Горутина-писатель заблокируется на `writerSem` пока оба читателя не вызовут `RUnlock()`. После второго `RUnlock()` `readerWait` станет 0, писатель проснётся и напечатает "writer in". Новые `RLock()` вызванные после `Lock()` блокируются на `readerSem` т.к. `readerCount < 0`.
![[sync.RWMutex#^rwmx-semaphore-roles]]

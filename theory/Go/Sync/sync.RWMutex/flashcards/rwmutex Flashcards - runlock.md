#flashcards/rwmutex/runlock

Опиши fast path и slow path в `RUnlock()`. Что является триггером для slow path?
?
![[RUnlock()#^runlock-fast-path]]
![[RUnlock()#^runlock-slow-path]]

Как последний из "старых" читателей определяет что именно он должен разбудить писателя?
?
![[RUnlock()#^runlock-last-reader]]

`rUnlockSlow` проверяет два условия для вызова `fatal`. Что такое `r+1` и что каждое условие обнаруживает?
?
![[RUnlock()#^runlock-fatal-logic]]
![[RUnlock()#^runlock-fatal-cases]]

`readerWait` декрементируется в `rUnlockSlow` только если `readerCount < 0`. Почему не всегда?
?
`readerWait` имеет смысл только когда писатель ждёт. Если писателя нет (`readerCount >= 0`), `rUnlockSlow` вообще не вызывается — `RUnlock()` уходит по fast path. Декрементировать `readerWait` без активного писателя бессмысленно и потенциально опасно — следующий `Lock()` получит некорректное начальное значение.
![[RUnlock()#^runlock-fast-path]]

Что выведет этот код?
```go
var rw sync.RWMutex
rw.RUnlock() // паника?
```
?
`fatal error: sync: RUnlock of unlocked RWMutex` — вызов `rUnlockSlow` с `r = -1`, `r+1 = 0` → первое условие срабатывает.
![[RUnlock()#^runlock-fatal-cases]]

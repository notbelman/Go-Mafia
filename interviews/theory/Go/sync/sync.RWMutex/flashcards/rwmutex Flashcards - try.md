#flashcards/rwmutex/try

Чем `TryRLock()` отличается от `RLock()` по поведению и реализации?
?
![[TryRLock_TryLock#^try-rlock-def]]
![[TryRLock_TryLock#^try-rlock-impl]]

Опиши алгоритм `TryLock()` по шагам. Почему он двухэтапный?
?
![[TryRLock_TryLock#^try-lock-def]]
![[TryRLock_TryLock#^try-lock-impl]]

`TryLock()` захватил внутренний мьютекс `w`, но `readerCount != 0`. Почему нельзя просто вернуть `false` не трогая `w`?
?
![[TryRLock_TryLock#^try-lock-rollback]]

Документация Go предупреждает об использовании `TryLock`. Что именно она говорит и почему?
?
![[TryRLock_TryLock#^try-when-to-use]]

В каких сценариях `TryLock`/`TryRLock` оправданы?
?
![[TryRLock_TryLock#^try-use-cases]]

Что выведет этот код?
```go
var rw sync.RWMutex
rw.RLock()
fmt.Println(rw.TryLock())
fmt.Println(rw.TryRLock())
rw.RUnlock()
fmt.Println(rw.TryLock())
```
?
`false` — есть активный читатель, `TryLock` не может взять write lock.
`true` — `TryRLock` успешен, читателей может быть несколько.
`true` — после `RUnlock()` `readerCount = 0` (но `TryRLock` добавил 1, нужно `RUnlock()` для него тоже). Точнее: после RUnlock первого, readerCount=1 (TryRLock добавил). TryLock видит readerCount=1 → false. Нужно сначала освободить TryRLock. На практике: первый false, второй true, третий зависит от того освобождён ли TryRLock — здесь он не освобождён → false.
![[TryRLock_TryLock#^try-rlock-impl]]
![[TryRLock_TryLock#^try-lock-impl]]

**TryRLock** — пытается взять read lock без блокировки. ^try-rlock-def

- Возвращает `true` если получилось, `false` если есть писатель
- Использует CAS в цикле: `readerCount.CompareAndSwap(c, c+1)`
- Если `readerCount < 0` → сразу `return false`

^try-rlock-impl

**TryLock** — пытается взять write lock без блокировки. ^try-lock-def

- Сначала `w.TryLock()` — если не получилось, `return false`
- Потом `readerCount.CompareAndSwap(0, -rwmutexMaxReaders)`
- Если есть хоть один читатель → **откатывает** `w.Unlock()` и `return false`

^try-lock-impl

## Почему TryLock откатывает w.Unlock() при наличии читателей?

`TryLock` сначала захватывает внутренний мьютекс `w`, потом пытается атомарно поставить `readerCount` в `-rwmutexMaxReaders`. Если `readerCount != 0` (есть читатели), CAS провалится. Нельзя держать `w` заблокированным и вернуть `false` — это заблокирует других писателей. Поэтому откатываем `w.Unlock()`. ^try-lock-rollback

## Когда использовать?

Редко. Документация Go прямо предупреждает: правильные применения TryLock существуют, но они редки, и использование TryLock часто признак более глубокой проблемы. ^try-when-to-use

Типичные случаи: ^try-use-cases

- Optimistic locking / fallback логика
- Избежание deadlock в сложных сценариях
- "Попробуй, если не получится — делай что-то другое"

## Связь
- [[sync.RWMutex]] — структура
- [[RLock()]] — обычный RLock vs TryRLock
- [[sync.Mutex.TryLock]] — аналогичная идея для Mutex

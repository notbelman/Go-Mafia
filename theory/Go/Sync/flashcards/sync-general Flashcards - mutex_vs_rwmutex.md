#flashcards/sync-general/mutex_vs_rwmutex

Почему каждый вызов RLock() вызывает cache invalidation на других ядрах?
?
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-rlock-atomic]]

Что такое cache invalidation и как она связана с RWMutex?
?
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-cache-invalidation]]

Насколько медленнее чтение из RAM по сравнению с чтением из кэша CPU?
?
![[sync.Mutex_vs_sync.RWMutex#^cache-vs-ram-latency]]

8 горутин на 8 ядрах одновременно делают RLock(). Что происходит с производительностью и почему?
?
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-contention-cascade]]

Почему Mutex не вызывает cache invalidation у ждущих горутин?
?
![[sync.Mutex_vs_sync.RWMutex#^mutex-no-cache-invalidation-when-locked]]

Как работает Mutex.Lock() на уровне CPU — что такое CAS?
?
![[sync.Mutex_vs_sync.RWMutex#^mutex-cas]]

При каком условии RWMutex выигрывает у Mutex? Сформулируй правило.
?
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-win-condition]]

Покажи пример где RWMutex плохо подходит (overhead не окупается) и где хорошо. Конкретные числа.
?
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-tradeoff-examples]]

Перечисли 4 условия, при которых нужно использовать Mutex по умолчанию.
?
![[sync.Mutex_vs_sync.RWMutex#^mutex-when-to-use]]

Перечисли 4 условия, при которых RWMutex даёт реальный выигрыш.
?
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-when-to-use]]

Что выведет этот код — и быстро или медленно он работает на 8 ядрах?
```go
var mu sync.RWMutex
var m = map[string]int{"key": 42}

func readValue() int {
    mu.RLock()
    defer mu.RUnlock()
    return m["key"]  // ~10ns
}
```
?
Код корректен, но производительность хуже Mutex при высоком параллелизме. `RLock()` делает `atomic.Add(&readerCount, 1)` — каждый вызов инвалидирует кэш на всех других ядрах. Критическая секция 10ns, а overhead RLock ~100ns — не окупается.
![[sync.Mutex_vs_sync.RWMutex#^rwmutex-tradeoff-examples]]

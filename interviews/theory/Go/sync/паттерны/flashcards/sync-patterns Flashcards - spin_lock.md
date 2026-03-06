#flashcards/sync-patterns/spin_lock

Что такое spin lock? Опиши механизм работы.
?
![[Spin_lock#^spinlock-impl]]

Что такое busy waiting? Чем отличается от park?
?
![[Spin_lock#^busy-waiting-def]]

При каких условиях busy waiting оправдан? При каких — нет?
?
![[Spin_lock#^busy-waiting-when-ok]]
![[Spin_lock#^busy-waiting-when-bad]]

Что делает runtime.Gosched() в спинлоке? Что будет без него?
?
![[Spin_lock#^gosched-effect]]

Назови три проблемы spin lock.
?
![[Spin_lock#^spinlock-no-fairness]]
![[Spin_lock#^spinlock-cache-bouncing]]
![[Spin_lock#^spinlock-scalability]]

Что такое TTAS оптимизация? Почему она снижает инвалидации кэша?
?
![[Spin_lock#^ttas-explanation]]

Почему spin lock хорош для критических секций < 1 мкс?
?
![[Spin_lock#^busy-waiting-when-ok]]

Что выведет этот код при запуске на 4 CPU?
```go
type SpinLock struct{ locked atomic.Bool }
func (s *SpinLock) Lock() {
    for !s.locked.CompareAndSwap(false, true) {
        runtime.Gosched()
    }
}
func (s *SpinLock) Unlock() { s.locked.Store(false) }

var mu SpinLock
var count int
var wg sync.WaitGroup
for i := 0; i < 1000; i++ {
    wg.Add(1)
    go func() { defer wg.Done(); mu.Lock(); count++; mu.Unlock() }()
}
wg.Wait()
fmt.Println(count)
```
?
`1000` — spin lock корректно защищает критическую секцию. Несмотря на гонку горутин за лок, каждый инкремент происходит под защитой CAS. Но при 1000 горутинах высокий contention — в реальных системах здесь лучше sync.Mutex.
![[Spin_lock#^spinlock-impl]]

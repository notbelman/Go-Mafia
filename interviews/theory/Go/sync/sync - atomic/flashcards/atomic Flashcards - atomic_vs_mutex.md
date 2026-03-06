#flashcards/atomic/atomic_vs_mutex

Что защищает atomic, а что — mutex? В чём принципиальная разница?
?
![[Atomic vs Mutex#^atomic-vs-mutex-table]]

Какую одну CPU инструкцию использует atomic для Add? Для CompareAndSwap? Для Swap?
?
![[Atomic vs Mutex#^atomic-lock-xadd]]

Что делает префикс LOCK в CPU-инструкциях atomic?
?
![[Atomic vs Mutex#^atomic-lock-prefix]]

Из каких компонентов состоит Mutex под капотом? Назови все 4.
?
![[Atomic vs Mutex#^mutex-under-hood]]

Сколько циклов спиннинга делает Mutex перед тем как усыпить горутину?
?
![[Atomic vs Mutex#^mutex-spinning]]

Что такое futex и для чего он используется в Mutex?
?
![[Atomic vs Mutex#^mutex-futex]]

Что происходит с горутинами при конкуренции на atomic vs при конкуренции на mutex?
?
![[Atomic vs Mutex#^atomic-vs-mutex-table]]

Назови 3 типичных юзкейса для atomic и для mutex.
?
![[Atomic vs Mutex#^atomic-vs-mutex-table]]

Что выведет этот код?
```go
var counter atomic.Int64
var wg sync.WaitGroup
for i := 0; i < 1000; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        counter.Add(1)
    }()
}
wg.Wait()
fmt.Println(counter.Load())
```
?
`1000` — каждый `Add` атомарен, нет гонки. Все 1000 инкрементов гарантированно применяются.
![[Atomic vs Mutex#^atomic-guarantee]]

Что выведет этот код и почему это проблема?
```go
var mu sync.Mutex
var counter int
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        mu.Lock()
        counter++
        mu.Unlock()
    }()
}
wg.Wait()
fmt.Println(counter)
```
?
`5` — mutex корректно защищает инкремент. Это правильный паттерн для non-atomic операции над полем структуры.
![[Atomic vs Mutex#^mutex-guarantee]]

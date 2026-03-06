#flashcards/mutex/trylock

В какой версии Go появился TryLock?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-version]]

Что возвращает TryLock и что нужно сделать при true?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-what]]

При каких двух условиях TryLock возвращает false? Есть ли неочевидный кейс?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-false-locked]] + ![[sync.Mutex.TryLock (Go 1.18+)#^trylock-false-starving]]

Почему TryLock возвращает false в Starvation mode даже если мьютекс свободен?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-false-starving]]

Как реализован TryLock внутри? Какие операции используются?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-impl]]

Какие memory model гарантии у успешного TryLock? У неудачного?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-memmodel-success]] + ![[sync.Mutex.TryLock (Go 1.18+)#^trylock-memmodel-fail]]

Что говорит официальная документация Go об использовании TryLock?
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-warning]]

Назови три легитимных кейса использования TryLock.
?
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-usecase-pool]] + ![[sync.Mutex.TryLock (Go 1.18+)#^trylock-usecase-deadlock]] + ![[sync.Mutex.TryLock (Go 1.18+)#^trylock-usecase-optimistic]]

Что произойдёт если использовать TryLock в цикле как замену Lock()?
?
Livelock — горутины будут постоянно пробовать и проигрывать, не прогрессируя. TryLock в цикле — антипаттерн.
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-warning]]

Что выведет этот код?
```go
var mu sync.Mutex
mu.Lock()
fmt.Println(mu.TryLock()) // ?
mu.Unlock()
fmt.Println(mu.TryLock()) // ?
```
?
`false` затем `true` — первый TryLock возвращает false т.к. мьютекс захвачен. После Unlock() мьютекс свободен, второй TryLock успешен. После второго TryLock нужен Unlock() — мьютекс захвачен.
![[sync.Mutex.TryLock (Go 1.18+)#^trylock-what]]

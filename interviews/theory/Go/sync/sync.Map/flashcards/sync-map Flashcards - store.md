#flashcards/sync-map/store

Опиши алгоритм Store() по шагам. Когда берётся лок, когда нет?
?
![[Store#^store-algorithm]]

Когда Store() работает lock-free?
?
![[Store#^store-lockfree-condition]]

Почему Store() использует CAS в trySwap, а не просто atomic.Store?
?
![[Store#^store-cas-reason]]

Что такое "double-check после лока" в Store()? Зачем это нужно?
?
![[Store#^store-double-check]]

Store нового ключа (нет ни в read, ни в dirty). Опиши все шаги.
?
![[Store#^store-new-key]]

entry.p == expunged. Вызываем Store на этот ключ. Что произойдёт?
?
![[Store#^store-algorithm]]

Что выведет этот код?
```go
var m sync.Map
m.Store("a", 1)
m.Store("a", 2)
val, _ := m.Load("a")
fmt.Println(val)
```
?
`2` — второй Store обновляет существующий ключ. Если ключ уже в read и не expunged — CAS атомарно обновляет entry.p без лока.
![[Store#^store-lockfree-condition]]

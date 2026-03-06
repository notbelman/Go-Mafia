#flashcards/sync-map/methods

Какие базовые методы sync.Map lock-free (при hit в read)?
?
![[Все_методы#^methods-basic-table]]

Clear() — lock-free или нет?
?
![[Все_методы#^methods-basic-table]]

Что делает LoadOrStore? В чём отличие от Store?
?
![[Все_методы#^methods-combined-table]]

Что делает LoadAndDelete?
?
![[Все_методы#^methods-combined-table]]

Что делает Swap?
?
![[Все_методы#^methods-combined-table]]

Что делает CompareAndSwap? С какой версии Go доступен?
?
![[Все_методы#^methods-conditional-table]]

Что делает CompareAndDelete? С какой версии Go доступен?
?
![[Все_методы#^methods-conditional-table]]

Как остановить итерацию в Range?
?
![[Все_методы#^methods-range-table]]

Что выведет этот код?
```go
var m sync.Map
m.Store("x", 1)
actual, loaded := m.LoadOrStore("x", 99)
fmt.Println(actual, loaded)
actual2, loaded2 := m.LoadOrStore("y", 42)
fmt.Println(actual2, loaded2)
```
?
`1 true` — LoadOrStore вернул существующее значение, loaded=true.
`42 false` — ключа "y" не было, сохранил 42, loaded=false.
![[Все_методы#^methods-combined-table]]

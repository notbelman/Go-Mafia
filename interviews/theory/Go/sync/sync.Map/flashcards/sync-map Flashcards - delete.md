#flashcards/sync-map/delete

Опиши алгоритм Delete() по шагам.
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Map/Delete#^delete-algorithm]]

Delete НЕ удаляет запись из map. Что именно происходит?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Map/Delete#^delete-soft]]

Когда происходит физическое удаление записи из map?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Map/Delete#^delete-physical]]

Ключ есть в read. Вызываем Delete. Берётся ли mutex?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Map/Delete#^delete-algorithm]]

Что выведет этот код?
```go
var m sync.Map
m.Store("a", 1)
m.Delete("a")
val, ok := m.Load("a")
fmt.Println(val, ok)
```
?
`<nil> false` — после Delete entry.p = nil, Load проверяет nil и возвращает nil, false. Запись физически остаётся в map, но считается удалённой.
![[WORK-BASE/interviews/theory/Go/sync/sync.Map/Delete#^delete-soft]]

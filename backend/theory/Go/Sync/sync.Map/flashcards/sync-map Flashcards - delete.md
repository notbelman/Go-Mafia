#flashcards/sync-map/delete

Опиши алгоритм Delete() по шагам.
?
![[Delete()#^delete-algorithm]]

Delete НЕ удаляет запись из map. Что именно происходит?
?
![[Delete()#^delete-soft]]

Когда происходит физическое удаление записи из map?
?
![[Delete()#^delete-physical]]

Ключ есть в read. Вызываем Delete. Берётся ли mutex?
?
![[Delete()#^delete-algorithm]]

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
![[Delete()#^delete-soft]]

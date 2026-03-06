#flashcards/map/set

Как реализовать set в Go? Покажи паттерн.
?
![[Set в Go#^set-pattern]]

Почему для set используют `map[T]struct{}`, а не `map[T]bool`? Конкретные числа.
?
![[Set в Go#^set-struct-zero]]

Что выведет этот код?
```go
set := map[string]struct{}{}
set["apple"] = struct{}{}
set["banana"] = struct{}{}
delete(set, "apple")
_, ok := set["apple"]
fmt.Println(ok, len(set))
```
?
`false 1` — "apple" удалён, len = 1. Операции добавления, проверки и удаления работают стандартными операторами map.
![[Set в Go#^set-code]]

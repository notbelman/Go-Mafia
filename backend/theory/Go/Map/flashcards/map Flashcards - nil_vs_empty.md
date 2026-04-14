#flashcards/map/nil_vs_empty

В чём разница между `var m map[string]int` и `m := map[string]int{}`?
?
![[nil vs empty map#^799dc9]]

Какое главное правило про nil map и операции с ней?
?
![[nil vs empty map#^nil-read-only]]

Что произойдёт при `delete(nilMap, "key")`?
?
![[nil vs empty map#^nil-ops-table]]

Что произойдёт при `len(nilMap)`?
?
![[nil vs empty map#^nil-ops-table]]

Что произойдёт при `for range nilMap`?
?
![[nil vs empty map#^nil-ops-table]]

Что произойдёт при `nilMap["key"] = 1`?
?
![[nil vs empty map#^nil-ops-table]]

Чем отличается JSON-сериализация nil map от empty map?
?
![[nil vs empty map#^nil-json-null]]

Когда JSON-разница nil/empty map критична на практике?
?
![[nil vs empty map#^nil-json-advice]]

Что выведет этот код?
```go
var m map[string]int
fmt.Println(m["missing"])
fmt.Println(len(m))
m["key"] = 1
```
?
Выведет `0` и `0`, затем паника: `assignment to entry in nil map`. Читать из nil map безопасно — возвращает zero value. Писать — panic.
![[nil vs empty map#^nil-ops-table]]

Что выведет этот код?
```go
import "encoding/json"

var nilMap map[string]int
emptyMap := map[string]int{}
b1, _ := json.Marshal(nilMap)
b2, _ := json.Marshal(emptyMap)
fmt.Println(string(b1), string(b2))
```
?
`null {}` — nil map сериализуется в JSON null, empty map в пустой объект `{}`. Критично когда API ожидает `{}`.
![[nil vs empty map#^nil-json-null]]

#flashcards/atomic/atomic_value

Что хранит atomic.Value внутри? Опиши внутреннюю структуру.
?
![[atomic.Value#^value-internals]]

Как называется внутренняя структура atomic.Value и какие у неё поля?
?
![[atomic.Value#^value-struct]]

Что произойдёт при `v.Store(nil)`? Почему?
?
![[atomic.Value#^value-nil-panic]]

Что произойдёт если сначала сделать `v.Store("hello")`, а потом `v.Store(42)`?
?
![[atomic.Value#^value-type-consistency]]

Почему нельзя хранить разные типы в одном atomic.Value? Какой механизм это обеспечивает?
?
![[atomic.Value#^value-type-consistency]]

Что произойдёт если скопировать atomic.Value после Store?
?
![[atomic.Value#^value-nocopy]]

Перечисли три правила atomic.Value и что нарушается при каждом.
?
![[atomic.Value#^value-rules]]

Что выведет этот код?
```go
var v atomic.Value
v.Store("config-v1")
v.Store("config-v2")
fmt.Println(v.Load())
```
?
`config-v2` — оба Store одного типа (string), это разрешено. Load возвращает последнее сохранённое значение.
![[atomic.Value#^value-type-consistency]]

Что произойдёт при запуске?
```go
var v atomic.Value
v.Store("hello")
v2 := v
fmt.Println(v2.Load())
```
?
Undefined behavior — копирование atomic.Value после Store запрещено (noCopy). `go vet` поймает это как ошибку.
![[atomic.Value#^value-nocopy]]

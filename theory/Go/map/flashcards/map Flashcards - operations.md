#flashcards/map/operations

Опиши алгоритм чтения `v := m[key]` по шагам
?
![[операции чтения и вставки#^op-read-steps]]

Что вернёт чтение из nil map? Безопасно ли это?
?
![[операции чтения и вставки#^op-nil-read-safe]]

Опиши алгоритм вставки `m[key] = value` по шагам
?
![[операции чтения и вставки#^op-write-steps]]

Что происходит при вставке в nil map?
?
![[операции чтения и вставки#^op-write-steps]]

Что происходит при вставке во время resize?
?
![[операции чтения и вставки#^op-write-steps]]

Что происходит если при вставке нет пустых слотов в bucket и всей цепочке overflow?
?
![[операции чтения и вставки#^op-write-steps]]

При каком load factor (конкретное число) вставка триггерит resize в Go?
?
![[операции чтения и вставки#^op-write-steps]]

Как работает `delete(m, key)` внутри? Что происходит с памятью bucket?
?
![[операции чтения и вставки#^op-delete-no-free]]

Что выведет этот код?
```go
var m map[string]int
fmt.Println(m["key"])
fmt.Println(len(m))
```
?
`0` и `0` — чтение из nil map безопасно, возвращает zero value. len(nil map) = 0.
![[операции чтения и вставки#^op-nil-read-safe]]

Что произойдёт?
```go
var m map[string]int
m["key"] = 1
```
?
`panic: assignment to entry in nil map` — запись в nil map паникует. Нужно сначала инициализировать через make или литерал.
![[операции чтения и вставки#^op-write-steps]]

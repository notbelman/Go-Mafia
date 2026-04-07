#flashcards/map/addr_value

Что произойдёт при компиляции этого кода?
```go
m := map[string]User{"bob": {Age: 30}}
m["bob"].Age = 31
```
?
Ошибка компиляции: `cannot assign to struct field m["bob"].Age in map`. Взять адрес value в map нельзя.
![[нельзя взять адрес value#^addr-compile-error]]

Почему в Go нельзя взять адрес value из map?
?
![[нельзя взять адрес value#^addr-why-evacuation]]

Покажи на примере, что происходит с указателем на value после resize
?
![[нельзя взять адрес value#^addr-invalidation-example]]

Назови два способа обойти ограничение "нельзя изменить поле struct в map"
?
![[нельзя взять адрес value#^addr-solutions]]

В чём разница между `map[string]User` и `map[string]*User` с точки зрения изменения полей?
?
![[нельзя взять адрес value#^addr-solutions]]

Что выведет этот код?
```go
type User struct{ Age int }
m := map[string]*User{"bob": {Age: 30}}
m["bob"].Age = 31
fmt.Println(m["bob"].Age)
```
?
`31` — храним указатель, адрес User в памяти не меняется при resize, меняется только запись в buckets. Изменение через разыменование работает.
![[нельзя взять адрес value#^addr-solutions]]

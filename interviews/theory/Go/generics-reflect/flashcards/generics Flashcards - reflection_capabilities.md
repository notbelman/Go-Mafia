#flashcards/generics/reflection_capabilities

Какие 6 групп операций позволяет делать пакет reflect?
?
![[Возможности_рефлексии#^refl-analysis]]
![[Возможности_рефлексии#^refl-create-types]]
![[Возможности_рефлексии#^refl-call]]
![[Возможности_рефлексии#^refl-checks]]
![[Возможности_рефлексии#^refl-chan-select]]
![[Возможности_рефлексии#^refl-nil-invalid]]

Какие методы reflect.Type используются для анализа полей и методов структуры?
?
![[Возможности_рефлексии#^refl-analysis]]

Сколько возвращает NumIn() для метода с одним параметром, и почему?
?
![[Возможности_рефлексии#^refl-numin-receiver]]

Как вызвать метод через рефлексию? Нужно ли передавать ресивер в args?
?
![[Возможности_рефлексии#^refl-methodbyname]]

Что выведет этот код?
```go
type Vec struct{ X, Y int }
func (v Vec) Add(other Vec) Vec { return Vec{v.X + other.X, v.Y + other.Y} }

v := reflect.ValueOf(Vec{1, 2})
result := v.MethodByName("Add").Call([]reflect.Value{reflect.ValueOf(Vec{3, 4})})
fmt.Println(result[0])
```
?
`{4 6}` — `MethodByName` уже связывает ресивер `Vec{1,2}`, в args передаём только `other`. Результат — срез `[]reflect.Value`, берём `result[0]`.
![[Возможности_рефлексии#^refl-methodbyname]]

Как получить reflect.Type интерфейса для проверки Implements? Почему используется nil?
?
![[Возможности_рефлексии#^refl-implements-nil]]

Почему reflect.Select предпочтительнее обычного select в некоторых случаях?
?
![[Возможности_рефлексии#^refl-select-dynamic]]

Как создать тип структуры динамически через reflect? Какие функции используются для массива и указателя?
?
![[Возможности_рефлексии#^refl-create-types]]
![[Возможности_рефлексии#^refl-makechan]]

Что выведет этот код?
```go
v := reflect.ValueOf(nil)
fmt.Println(v.IsValid())
fmt.Println(v.Kind())
```
?
`false` и `invalid` — nil превращается в нулевое `reflect.Value`. `IsValid()` возвращает false. Вызов любого другого метода (кроме `Kind`, `String`) вызвал бы панику.
![[Возможности_рефлексии#^refl-nil-invalid]]
![[Возможности_рефлексии#^refl-invalid-panic]]

Что произойдёт при вызове любого метода на невалидном reflect.Value, кроме IsValid и Kind?
?
![[Возможности_рефлексии#^refl-invalid-panic]]

Как сделать select с динамическим (неизвестным на этапе компиляции) количеством каналов?
?
![[Возможности_рефлексии#^refl-chan-select]]
![[Возможности_рефлексии#^refl-select-dynamic]]

#flashcards/generics/limitations

Можно ли объявить метод с собственными type-параметрами в Go? Как обойти это ограничение?
?
![[Ограничения дженериков#^no-generic-methods-workaround]]

Что выведет этот код — скомпилируется ли он?
```go
type MyStruct struct{}

func (s *MyStruct) Process[T any](v T) {
    fmt.Println(v)
}

func main() {
    s := &MyStruct{}
    s.Process(42)
}
```
?
Не скомпилируется — в Go нельзя объявить метод с собственными type-параметрами. Обходной путь: `func Process[T any](s *MyStruct, v T)`.
![[Ограничения дженериков#^no-generic-methods-workaround]]

Почему нельзя создать константу типа `T` в generic-функции, даже если constraint — один конкретный тип?
?
![[Ограничения дженериков#^no-generic-const]]

В чём принципиальное отличие Go от C++ в части передачи числовых значений как параметров шаблонов/типов?
?
![[Ограничения дженериков#^no-nttp-values]]

Что из этого работает — embedding или именованное поле?
```go
type Wrapper[T any] struct {
    T        // (1)
    Value T  // (2)
}
```
?
![[Ограничения дженериков#^no-embed-generic]]

Скомпилируется ли этот код? Если нет — как исправить?
```go
type NumConstraint interface { int32 | int64 }
func process[T *NumConstraint](v T) { fmt.Println(v) }
```
?
![[Ограничения дженериков#^pointer-constraint-workaround]]

Почему нельзя объединить map и slice в одном constraint для индексации? Что именно несовместимо?
?
![[Ограничения дженериков#^no-map-slice-union]]

Скомпилируется ли этот код?
```go
func first[T ~string | ~[]byte](v T) byte {
    return v[0]
}
```
?
Да — чтение по индексу у `string` и `[]byte` совместимо. Обе операции возвращают `byte`.
![[Ограничения дженериков#^string-byte-readonly]]

Скомпилируется ли этот код?
```go
func setFirst[T ~string | ~[]byte](v T) {
    v[0] = 'x'
}
```
?
Нет — `string` иммутабельна, запись `v[0] = 'x'` не компилируется для constraint включающего `~string`.
![[Ограничения дженериков#^string-byte-readonly]]

Почему `v.Bar()` не компилируется, хотя реальный тип `S` имеет метод `Bar`?
```go
type C interface { ~struct{}; Foo() }
func process[T C](v T) {
    v.Foo()  // ok
    v.Bar()  // ???
}
```
?
![[Ограничения дженериков#^method-not-in-constraint]]

Перечисли все основные ограничения дженериков в Go (6 штук из tldr).
?
![[Ограничения дженериков#^no-generic-methods-workaround]] + ![[Ограничения дженериков#^no-generic-const]] + ![[Ограничения дженериков#^no-nttp-values]] + ![[Ограничения дженериков#^no-embed-generic]] + ![[Ограничения дженериков#^pointer-constraint-workaround]] + ![[Ограничения дженериков#^method-not-in-constraint]]

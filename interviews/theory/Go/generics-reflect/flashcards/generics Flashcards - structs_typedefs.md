#flashcards/generics/structs_typedefs

Как правильно объявить метод на обобщённой структуре? Чем отличается ресивер от обычного?
?
![[Обобщённые структуры и type definitions#^generic-struct-receiver]]

Что выведет этот код и почему он компилируется?
```go
type Set[T comparable] struct{ m map[T]struct{} }

func (s *Set[T]) Add(v T)     { s.m[v] = struct{}{} }
func (s *Set[_]) Len() int    { return len(s.m) }

s := &Set[string]{m: make(map[string]struct{})}
s.Add("hello")
s.Add("world")
fmt.Println(s.Len())
```
?
`2` — `Add` использует `T` из ресивера `Set[T]`, `Len` не использует тип параметра и пишет `_`. Оба метода валидны.
![[Обобщённые структуры и type definitions#^generic-struct-receiver]]

В чём разница между полным и частичным инстанцированием обобщённого типа? Через что можно сделать частичное?
?
![[Обобщённые структуры и type definitions#^generic-typedef-partial]]

Что из этого скомпилируется до Go 1.24?
```go
type Pair[T, U any] struct{ First T; Second U }
type A = Pair[string, int]          // (1)
type B[U any] Pair[string, U]       // (2)
type C[U any] = Pair[string, U]     // (3)
```
?
`(1)` и `(2)` — полное инстанцирование через alias и частичное через type definition. `(3)` не компилируется до Go 1.24 — обобщённые aliases не поддерживались.
![[Обобщённые структуры и type definitions#^generic-alias-go124]] + ![[Обобщённые структуры и type definitions#^generic-typedef-partial]]

С какой версии Go поддерживаются обобщённые type aliases?
?
![[Обобщённые структуры и type definitions#^generic-alias-go124]]

Зачем прятать сложный обобщённый тип за alias или частичное инстанцирование? Приведи пример.
?
![[Обобщённые структуры и type definitions#^generic-hide-complexity]]

Что выведет этот код?
```go
func printAll[T any](vals ...T) {
    for _, v := range vals { fmt.Println(v) }
}
printAll(10, 20, 30)
```
?
`10`, `20`, `30` — по одному на строку. `T` выводится как `int` из первого аргумента. Variadic с дженериком требует всех аргументов одного типа.
![[Обобщённые структуры и type definitions#^generic-variadic]]

Можно ли передать в `printAll[T any](vals ...T)` аргументы разных типов? Почему?
?
![[Обобщённые структуры и type definitions#^generic-variadic]]

Что такое обобщённый constraint (template template parameter) в Go? Когда он нужен?
?
![[Обобщённые структуры и type definitions#^generic-constraint-template]]

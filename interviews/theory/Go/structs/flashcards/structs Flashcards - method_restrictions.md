#flashcards/structs/method_restrictions

Для каких четырёх категорий типов нельзя создавать методы в Go?
?
![[Ограничения методов и unsafe cast#^method-restrictions]]

Почему для `type AliasInt = int` нельзя создать методы, а для `type NewInt int` — можно?
?
![[Ограничения методов и unsafe cast#^method-typedef-ok]]

Можно ли создать методы для type definition указательного типа?
```go
type PtrDef *int
func (p PtrDef) Method() {}  // ?
```
?
![[Ограничения методов и unsafe cast#^method-restrictions-example]]

Для каких ещё типов помимо именованных struct можно сделать type definition и добавить методы?
?
![[Ограничения методов и unsafe cast#^method-typedef-funcs]]

Приведи пример type definition для func и map с методами.
?
![[Ограничения методов и unsafe cast#^typedef-examples]]

В чём трейдофф между `slowCast` и `fastCast` при конвертации `[]Integer` → `[]int`?
?
![[Ограничения методов и unsafe cast#^unsafe-cast-example]]

При каком условии unsafe cast между двумя типами срезов корректен?
?
![[Ограничения методов и unsafe cast#^unsafe-cast-condition]]

Что делает этот код и почему он работает?
```go
type Integer int
func fastCast(src []Integer) []int {
    return *(*[]int)(unsafe.Pointer(&src))
}
```
?
Конвертирует срез без копирования. Работает потому что `Integer` и `int` имеют одинаковый underlying type → идентичный memory layout. `unsafe.Pointer` обходит type system.
![[Ограничения методов и unsafe cast#^unsafe-cast-example]]

Как реализовать union (C-style) в Go через unsafe? Для чего это используется?
?
![[Ограничения методов и unsafe cast#^unsafe-unions]] + ![[Ограничения методов и unsafe cast#^unsafe-union-example]]

Сколько байт занимает Union в примере и какие типы можно хранить в нём одновременно?
?
![[Ограничения методов и unsafe cast#^unsafe-union-example]]

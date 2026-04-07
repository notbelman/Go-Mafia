#flashcards/structs/embedding_types

Как называется встроенное поле в Go и каково его имя?
?
![[Встраивание типов#^embt-anon-field]]

Что выведет этот код?
```go
type A struct{ Value int }
type B struct{ A }
type C struct{ *B }
c := C{B: &B{A: A{Value: 10}}}
fmt.Println(c.Value)
```
?
`10` — promotion позволяет пропустить несколько уровней. `c.Value` → `c.B.A.Value`.
![[Встраивание типов#^embt-multilevel]]

Почему нельзя рекурсивно встраивать структуру саму в себя? Что разрешено вместо этого?
?
![[Встраивание типов#^embt-no-recursive]]

Что выведет компилятор для `b.Value` если в `Base` встроены `D1` и `D2`, у обоих есть поле `Value`?
?
![[Встраивание типов#^embt-ambiguous-error]]

Как разрешить конфликт имён при встраивании нескольких типов с одинаковыми полями/методами?
?
![[Встраивание типов#^embt-name-conflict]]

Что выведет этот код?
```go
type Person struct{ Name string }
func (p Person) Intro() string { return "Mr/Mrs " + p.Name }
type Woman struct{ Person }
func (w Woman) Intro() string { return "Mrs " + w.Name }
w := Woman{Person: Person{Name: "Анна"}}
fmt.Println(w.Intro())
fmt.Println(w.Person.Intro())
```
?
`Mrs Анна` и `Mr/Mrs Анна` — метод `Woman.Intro` перекрывает `Person.Intro`, но Person.Intro доступен явно.
![[Встраивание типов#^embt-override]]

В чём разница между перекрытием (shadowing) и переопределением (override) при встраивании?
?
![[Встраивание типов#^embt-shadow-rule]]

Что случится если встроить интерфейс в структуру и не инициализировать поле-интерфейс?
?
![[Встраивание типов#^embt-interface-embed]]

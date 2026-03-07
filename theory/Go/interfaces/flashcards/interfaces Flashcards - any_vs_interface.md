#flashcards/interfaces/any_vs_interface

Чем `any` отличается от `interface{}` на уровне рантайма?
?
![[any vs interface{}#^any-interchangeable]]

С какой версии Go введён `any` и как он определён?
?
![[any vs interface{}#^any-alias-def]]

Где в исходниках Go определён `any`?
?
![[any vs interface{}#^any-source-alias]]

Какую структуру имеет `any` под капотом и сколько занимает байт?
?
![[any vs interface{}#^any-eface-size]]

Почему ввели `any` если `interface{}` уже существовал?
?
![[any vs interface{}#^any-readability]]

Почему опасно злоупотреблять `any` в сигнатурах функций?
?
![[any vs interface{}#^any-static-typing-loss]]

В каких двух сценариях `any` уместен как тип параметра?
?
![[any vs interface{}#^any-use-cases]]

Что выведет этот код?
```go
var a any = 42
var b interface{} = 42
a = b
b = a
fmt.Sprintf("%T %T", a, b)
```
?
`"int int"` — `any` и `interface{}` полностью взаимозаменяемы, присваивание в обе стороны работает без конверсий, тип внутри остаётся `int`.
![[any vs interface{}#^any-identical-behavior]]

Что плохого в таком контракте: `func (s *Storage) Get(id int) any`?
?
![[any vs interface{}#^any-use-cases]]

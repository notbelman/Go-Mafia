#flashcards/structs/type_alias_vs_definition

В чём разница между `type A = int` и `type A int`?
?
![[Type alias vs Type definition#^alias-syntax]]

Что произойдёт при компиляции? `type NewInt int; var x int = 42; var b NewInt = x`
?
![[Type alias vs Type definition#^alias-assignment]]

Что произойдёт при компиляции? `type AliasInt = int; var x int = 42; var a AliasInt = x`
?
![[Type alias vs Type definition#^alias-assignment]]

Почему для type definition методы базового типа недоступны, а поля — доступны?
?
![[Type alias vs Type definition#^alias-methods-why]]

Почему поля definition-типа видны без каста?
?
![[Type alias vs Type definition#^alias-fields-visible]]

Что выведет этот код?
```go
type Data struct{ Value int }
func (d Data) Print() { fmt.Println(d.Value) }
type NewData Data
nd := NewData{Value: 2}
nd.Print()
```
?
Ошибка компиляции: `nd.Print undefined (type NewData has no field or method Print)` — definition создаёт новый тип, методы базового не наследуются.
![[Type alias vs Type definition#^alias-methods-why]]

Что выведет этот код?
```go
type Data struct{ Value int }
func (d Data) Print() { fmt.Println(d.Value) }
type AliasData = Data
ad := AliasData{Value: 42}
ad.Print()
```
?
`42` — alias это тот же тип, методы доступны напрямую.
![[Type alias vs Type definition#^alias-assignment]]

Какой underlying type у `type B A` где `type A int`?
?
![[Type alias vs Type definition#^alias-underlying-chain]]

Почему alias не создаёт новый underlying type?
?
![[Type alias vs Type definition#^alias-underlying-rule]]

Назови два практических сценария использования type alias.
?
![[Type alias vs Type definition#^alias-use-cases]]

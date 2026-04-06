#flashcards/reflection/struct_tags

Что такое теги структур в Go? Каков их синтаксис?
?
![[Теги структур#^tags-syntax]]

Что означает тег `json:"-"`?
?
![[Теги структур#^tags-syntax]]

Может ли одно поле структуры иметь теги для нескольких пакетов одновременно? Как это работает?
?
![[Теги структур#^tags-multiple]]

Почему для чтения тегов используют `reflect.TypeOf`, а не `reflect.ValueOf`?
?
![[Теги структур#^tags-read-reflect]]

Что выведет этот код?
```go
type User struct {
    Email string `json:"email,omitempty" xml:"e"`
}
t := reflect.TypeOf(User{})
f := t.Field(0)
fmt.Println(f.Tag.Get("json"))
fmt.Println(f.Tag.Get("xml"))
fmt.Println(f.Tag.Get("db"))
```
?
`email,omitempty`, `e`, `` (пустая строка) — `Get` возвращает строку тега для ключа целиком, включая опции. Для несуществующего ключа возвращает `""`.
![[Теги структур#^tags-read-reflect]]
![[Теги структур#^tags-lookup-vs-get]]

В чём разница между `Tag.Get` и `Tag.Lookup`? Когда важно использовать `Lookup`?
?
![[Теги структур#^tags-lookup-vs-get]]

Что выведет этот код?
```go
type S struct {
    A string `json:""`
    B string
}
t := reflect.TypeOf(S{})
_, ok1 := t.Field(0).Tag.Lookup("json")
_, ok2 := t.Field(1).Tag.Lookup("json")
fmt.Println(ok1, ok2)
```
?
`true false` — поле `A` имеет тег `json` с пустым значением (`ok=true`), поле `B` не имеет тега `json` вообще (`ok=false`). `Get` для обоих вернул бы одинаковый `""` — в этом и ценность `Lookup`.
![[Теги структур#^tags-lookup-vs-get]]

Рефлексия возвращает значение тега как целую строку. Кто отвечает за парсинг опций вроде `omitempty`?
?
![[Теги структур#^tags-manual-parse]]

Как `encoding/json` разбирает тег `"email,omitempty"` чтобы выделить имя поля и опции?
?
![[Теги структур#^tags-manual-parse]]

Опиши внутренний цикл `json.Marshal` — какие типы рефлексии (`TypeOf`/`ValueOf`) используются и зачем оба нужны одновременно?
?
![[Теги структур#^tags-json-marshal-loop]]

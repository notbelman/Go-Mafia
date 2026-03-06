#flashcards/context/with_value

Для каких данных предназначен context.WithValue? Приведи примеры правильного и неправильного использования.
?
![[context WithValue#^wv-usecase]]
![[context WithValue#^wv-what-to-store]]

Какова сложность поиска значения в context.Value? Почему?
?
![[context WithValue#^wv-lookup-complexity]]

Что такое key shadowing в контексте и чем он опасен?
?
![[context WithValue#^wv-shadowing-danger]]
![[context WithValue#^wv-collision-timing]]

Почему ключи для context.WithValue должны быть неэкспортируемыми типами, а не строками?
?
![[context WithValue#^wv-unexported-key-why]]
![[context WithValue#^wv-collision-timing]]

Как Go сравнивает ключи в context.Value? Почему `type userIDKeyA struct{}` и `type userIDKeyB struct{}` не коллидируют?
?
![[context WithValue#^wv-interface-comparison]]

Что вернёт ctx.Value() если ключ не найден ни у кого в цепочке?
?
![[context WithValue#^wv-nil-not-panic]]

Какой overhead у context.WithValue при каждом вызове Value()?
?
![[context WithValue#^wv-overhead]]

Что выведет этот код?
```go
type keyA struct{}
type keyB struct{}

bg := context.Background()
ctx1 := context.WithValue(bg, keyA{}, "value-A")
ctx2 := context.WithValue(ctx1, keyB{}, "value-B")

fmt.Println(ctx2.Value(keyA{}))
fmt.Println(ctx2.Value(keyB{}))
fmt.Println(ctx1.Value(keyB{}))
```
?
`value-A`, `value-B`, `nil` — ctx2 хранит keyB и ходит вверх за keyA к ctx1. ctx1 ничего не знает про keyB, который добавили позже в дочернем узле.
![[context WithValue#^wv-lookup-complexity]]

Что выведет этот код?
```go
type key struct{}

bg := context.Background()
ctx1 := context.WithValue(bg, key{}, "original")
ctx2 := context.WithValue(ctx1, key{}, "shadowed")

fmt.Println(ctx1.Value(key{}))
fmt.Println(ctx2.Value(key{}))
```
?
`original`, `shadowed` — ctx2 нашёл своё значение и остановил поиск. ctx1 не знает про ctx2. Именно так работает shadowing: родитель не затронут, но ctx2 и все его потомки видят перекрытое значение.
![[context WithValue#^wv-shadowing-danger]]
![[context WithValue#^wv-interface-comparison]]

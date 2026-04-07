#flashcards/map/keys

Какие категории типов могут быть ключами map в Go?
?
![[Ключи map#^key-comparable-types]]

Какие три типа не могут быть ключами map и почему?
?
![[Ключи map#^key-forbidden-types]]

Можно ли использовать struct как ключ map? При каком условии — нельзя?
?
![[Ключи map#^key-struct]]

По чему сравниваются pointer-ключи в map — по адресу или по значению?
?
![[Ключи map#^key-pointer-semantics]]

Что выведет этот код?
```go
type Point struct{ X, Y int }
p1 := &Point{1, 2}
p2 := &Point{1, 2}
m := map[*Point]string{}
m[p1] = "first"
m[p2] = "second"
fmt.Println(len(m))
```
?
`2` — p1 и p2 это разные указатели (разные адреса), даже если значения одинаковые. Pointer-ключи сравниваются по адресу, не по содержимому.
![[Ключи map#^key-pointer-example]]

Что произойдёт если изменить объект, на который указывает pointer-ключ map?
?
![[Ключи map#^key-pointer-mutation]]

Что выведет этот код и почему?
```go
type Point struct{ X, Y int }
p := &Point{1, 2}
m := map[*Point]string{}
m[p] = "original"
p.X = 99
fmt.Println(m[p])
```
?
`"original"` — ключ это адрес указателя, он не изменился. Запись всё ещё находится по тому же адресу. Но логика сломана: ключ `p` теперь указывает на `{99, 2}`, хотя мы думали что он для `{1, 2}`.
![[Ключи map#^key-pointer-mutation]]

Что будет если использовать interface{} как ключ map, а внутри interface лежит []int?
?
![[Ключи map#^key-interface-panic]]

Почему компилятор не ловит ошибку с interface-ключом содержащим non-comparable тип?
?
![[Ключи map#^key-interface-panic]]

Что выведет этот код?
```go
m := map[interface{}]string{}
var k interface{} = []int{1, 2}
m[k] = "val"
fmt.Println("ok")
```
?
`panic: runtime error: hash of unhashable type []int` — компилятор не может проверить это статически (тип interface{}), паника происходит в runtime при хэшировании ключа.
![[Ключи map#^key-interface-panic]]

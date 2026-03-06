#flashcards/reflection/three_properties

Назови три свойства рефлексии в Go. Что описывает каждое?
?
![[Три свойства рефлексии#^prop1-implicit-conversion]]
![[Три свойства рефлексии#^prop2-interface-method]]
![[Три свойства рефлексии#^prop3-settable-rules]]

Что происходит при вызове `reflect.TypeOf(x)` и `reflect.ValueOf(x)`? Какое неявное преобразование происходит под капотом?
?
![[Три свойства рефлексии#^prop1-implicit-conversion]]

Что возвращает метод `Interface()` у `reflect.Value`? Зачем он нужен?
?
![[Три свойства рефлексии#^prop2-interface-method]]

Что выведет этот код?
```go
var x float64 = 3.14
v := reflect.ValueOf(x)
fmt.Println(v.CanSet())
```
?
`false` — `ValueOf` получает копию значения. Копия не адресуема, поэтому `CanSet()` возвращает `false`. Любой вызов `SetFloat` здесь вызовет панику.
![[Три свойства рефлексии#^prop3-settable-rules]]

Что выведет этот код?
```go
var x float64 = 3.14
v2 := reflect.ValueOf(&x)
fmt.Println(v2.CanSet())
v3 := reflect.ValueOf(&x).Elem()
fmt.Println(v3.CanSet())
```
?
`false` затем `true` — `ValueOf(&x)` — это копия указателя, сам указатель не settable. `.Elem()` разыменовывает его и даёт доступ к оригиналу `x`, который адресуем и settable.
![[Три свойства рефлексии#^prop3-settable-rules]]

Какое сообщение паники возникает при вызове `SetFloat` на non-settable значении?
?
![[Три свойства рефлексии#^prop3-panic-msg]]

Почему `ValueOf(&x)` не даёт `CanSet() == true`, хотя передаётся указатель?
?
![[Три свойства рефлексии#^prop3-analogy]]

Объясни аналогию между рефлексией и передачей аргументов в функцию для понимания settability.
?
![[Три свойства рефлексии#^prop3-analogy]]

Что выведет этот код?
```go
var z int = 42
var iface any = &z
v := reflect.ValueOf(iface)
v = v.Elem()
v = v.Elem()
v.SetInt(100)
fmt.Println(z)
```
?
`100` — первый `Elem()` разыменовывает интерфейсный слой (`any` → `*int`), второй разыменовывает указатель (`*int` → `int`). Итого два `Elem()` для значения, обёрнутого в интерфейс как указатель.
![[Три свойства рефлексии#^prop3-interface-double-elem]]

Почему при значении в интерфейсе нужно два `Elem()`, а не один?
?
![[Три свойства рефлексии#^prop3-interface-double-elem]]

Сравни три варианта: `ValueOf(x)`, `ValueOf(&x)`, `ValueOf(&x).Elem()`. Каков `CanSet` для каждого и почему?
?
![[Три свойства рефлексии#^prop3-settable-rules]]
![[Три свойства рефлексии#^prop3-analogy]]

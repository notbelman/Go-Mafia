#flashcards/map/pointer

Чем map отличается от slice и string с точки зрения того, что копируется при передаче в функцию?
?
![[Map = указатель#^map-ptr-sizes]]

Сколько байт занимает копия map при передаче в функцию? Почему?
?
![[Map = указатель#^map-ptr-sizes]]

Что произойдёт с оригинальной map если внутри функции добавить новый ключ?
?
![[Map = указатель#^map-ptr-pass]]

Что произойдёт с оригинальной map если внутри функции переприсвоить `m = newMap`?
?
![[Map = указатель#^map-ptr-reassign]]
![[Map = указатель#^map-ptr-reassign-why]]

Почему изменения видны снаружи при записи в map через функцию?
?
![[Map = указатель#^map-ptr-visible-why]]

Что такое zero value для map? Что случится при чтении из nil map? При записи?
?
![[Map = указатель#^map-ptr-nil]]

Что выведет этот код?
```go
func addKey(m map[string]int) {
    m["new"] = 42
}
func main() {
    m := map[string]int{"a": 1}
    addKey(m)
    fmt.Println(m)
}
```
?
`map[a:1 new:42]` — map передаётся как указатель на hmap, копируется только сам указатель (8 байт). Запись идёт в тот же hmap, изменения видны снаружи.
![[Map = указатель#^map-ptr-visible-why]]

Что выведет этот код?
```go
func replaceMap(m map[string]int) {
    m = map[string]int{"x": 100}
}
func main() {
    m := map[string]int{"a": 1}
    replaceMap(m)
    fmt.Println(m)
}
```
?
`map[a:1]` — переприсваивание меняет только локальную копию указателя внутри функции. Оригинальный указатель в main продолжает указывать на старый hmap.
![[Map = указатель#^map-ptr-reassign-why]]

Что выведет этот код?
```go
var m map[string]int
fmt.Println(m["key"])
m["key"] = 1
```
?
Напечатает `0`, затем panic: assignment to entry in nil map. Чтение из nil map возвращает zero value без паники, но запись в nil map — panic.
![[Map = указатель#^map-ptr-nil]]

Почему с map нет проблемы "потери append" как со slice при передаче в функцию?
?
![[Map = указатель#^map-ptr-vs-slice]]

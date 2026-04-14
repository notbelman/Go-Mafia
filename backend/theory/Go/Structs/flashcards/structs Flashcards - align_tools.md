#flashcards/structs/align_tools

Какие три функции unsafe позволяют инспектировать структуры? Что возвращает каждая?
?
![[Выравнивание инструменты и практика#^unsafe-funcs]]

На каком этапе известны результаты `unsafe.Sizeof`, `unsafe.Alignof`, `unsafe.Offsetof`?
?
![[Выравнивание инструменты и практика#^unsafe-compile-time]]

Что выведет этот код?
```go
type Data struct {
    A bool
    B int32
    C bool
}
d := Data{}
fmt.Println(unsafe.Sizeof(d), unsafe.Alignof(d), unsafe.Offsetof(d.B))
```
?
`12 4 4` — размер 12 (padding после A и C), выравнивание = max поле = int32 = 4, B начинается на offset 4.
![[Выравнивание инструменты и практика#^unsafe-funcs-example]]

Как сделать compile-time assert что структура ровно 64 байта? Почему нужно два выражения, а не одно?
?
![[Выравнивание инструменты и практика#^compile-assert-explanation]] + ![[Выравнивание инструменты и практика#^compile-assert-two-sides]]

Почему `var _ [unsafe.Sizeof(T{}) - 64]byte` не компилируется если T меньше 64 байт?
?
![[Выравнивание инструменты и практика#^compile-assert-explanation]]

Если размер структуры 65 байт, скомпилируется ли `var _ [unsafe.Sizeof(T{}) - 64]byte`? Что это значит для assert?
?
![[Выравнивание инструменты и практика#^compile-assert-two-sides]]

Как проверить compile-time что выравнивание структуры равно 8 и поле B на offset 4?
?
![[Выравнивание инструменты и практика#^compile-assert-align-offset]]

Где особенно важны compile-time проверки выравнивания и смещений?
?
![[Выравнивание инструменты и практика#^compile-assert-align-offset]]

Что делает утилита `fieldalignment` и как её запустить?
?
![[Выравнивание инструменты и практика#^fieldalignment-usage]]

В каких трёх сценариях оправдано оптимизировать порядок полей структуры?
?
![[Выравнивание инструменты и практика#^when-to-optimize]]

Какой принцип применять по умолчанию при проектировании структур?
?
![[Выравнивание инструменты и практика#^readability-first]]

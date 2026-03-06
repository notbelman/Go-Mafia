#flashcards/slice/not_comparable

Что произойдёт при компиляции: `a == b` где `a, b []int`?
?
![[Slice — не comparable#^cmp-no-eq]]

Можно ли сделать `map[[]int]bool`? Почему?
?
![[Slice — не comparable#^cmp-no-map-key]]

Какие два способа сравнить два слайса по содержимому? В чём разница по скорости и версии Go?
?
![[Slice — не comparable#^cmp-methods]]

Как `reflect.DeepEqual` ведёт себя при сравнении nil slice и empty slice?
?
![[Slice — не comparable#^cmp-deepequal-nil]]

Почему в Go не выбрали неявное сравнение слайсов по элементам или по указателю?
?
![[Slice — не comparable#^cmp-why-semantics]]

Почему слайс нельзя использовать как ключ мапы с точки зрения мутабельности? Почему массив — можно?
?
![[Slice — не comparable#^cmp-why-mutability]]

Что выведет этот код?
```go
var a []int
b := []int{}
fmt.Println(reflect.DeepEqual(a, b))
fmt.Println(a == nil, b == nil)
```
?
`false` — `reflect.DeepEqual` различает nil и empty slice. `true false` — `a` nil, `b` empty.
![[Slice — не comparable#^cmp-deepequal-nil]]

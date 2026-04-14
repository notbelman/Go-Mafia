#flashcards/slice/copy

Сколько элементов скопирует `copy(dst, src)` если len(dst)=3, len(src)=5?
?
![[Slice copy#^copy-min-len]]

Что вернёт `copy(dst, src)` если `dst = make([]int, 0, 10)` (len=0, cap=10)?
?
![[Slice copy#^copy-empty-dst]]

Что выведет этот код?
```go
src := []int{1, 2, 3, 4, 5}
dst := make([]int, 3)
n := copy(dst, src)
fmt.Println(n, dst)
```
?
`3 [1 2 3]` — скопировано min(3,5)=3 элемента.
![[Slice copy#^copy-min-len]]

Что выведет этот код?
```go
type User struct{ Name string }
src := []*User{{"Alice"}, {"Bob"}}
dst := make([]*User, len(src))
copy(dst, src)
dst[0].Name = "Changed"
fmt.Println(src[0].Name)
```
?
`Changed` — copy для указателей копирует сами указатели, не данные. src и dst указывают на те же объекты.
![[Slice copy#^copy-shallow-pointers]]

В чём разница shallow copy для примитивов vs указателей при использовании `copy`?
?
![[Slice copy#^copy-shallow-primitives]] + ![[Slice copy#^copy-shallow-pointers]]

Что выведет этот код?
```go
a := []int{1, 2, 3}
b := a
b[0] = 999
fmt.Println(a)
```
?
`[999 2 3]` — `b := a` копирует только дескриптор, оба slice шарят один underlying array.
![[Slice copy#^copy-assign-no-copy]]

Назови три способа сделать независимую копию слайса.
?
![[Slice copy#Как сделать независимую копию]]

Безопасен ли `copy(s[1:], s[:4])` при перекрытии срезов?
?
![[Slice copy#^copy-overlap-safe]]

Что выведет этот код?
```go
s := []int{1, 2, 3, 4, 5}
copy(s[1:], s[:4])
fmt.Println(s)
```
?
`[1 1 2 3 4]` — copy безопасно обрабатывает overlap, сдвигая элементы вправо.
![[Slice copy#^copy-overlap-safe]]

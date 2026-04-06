#flashcards/slice/subslice_bugs

При каком условии append на подсрез перезапишет данные оригинала?
?
![[Slice от slice баги#^slice-bug-overwrite-condition]]

Что выведет этот код?
```go
a := []int{1, 2, 3, 4, 5}
b := a[:2]
b = append(b, 99)
fmt.Println(a)
```
?
`[1 2 99 4 5]` — `cap(b) = 5 > len(b) = 2`, append не реаллоцирует и пишет 99 в позицию `a[2]` общего array.
![[Slice от slice баги#^slice-bug-overwrite]]

Что произойдёт с `sub` и `data` после этого кода? Почему?
```go
data := make([]int, 4, 6)
sub := data[2:6]
sub = append(sub, 1)
```
?
![[Slice от slice баги#^slice-bug-diverge]]

При реаллокации подсреза — от чего считается новый cap? От len подсреза или от cap оригинала?
?
![[Slice от slice баги#^slice-bug-diverge-newcap]]

Что выведет этот код?
```go
data := make([]int, 3, 6)
sub := data[1:3]
sub = append(sub, 77)
fmt.Println(len(data), data[3])
```
?
`3 77` — append записал 77 в `data[3]` (cap хватало), но `len(data)` не изменился. Данные в array есть, но data их "не видит".
![[Slice от slice баги#^slice-bug-independent-descriptors]]

Почему изменение `len` у одного среза не влияет на `len` другого среза с тем же underlying array?
?
![[Slice от slice баги#^slice-bug-independent-why]]

Как full slice expression защищает от бага с перезаписью? Какой синтаксис?
?
![[Slice от slice баги#^slice-fix-full-slice-syntax]]

Что выведет этот код?
```go
a := []int{1, 2, 3, 4, 5}
b := a[:2:2]
b = append(b, 99)
fmt.Println(a)
```
?
`[1 2 3 4 5]` — `cap(b) = 2 = len(b)`, append вынужден реаллоцировать, `b` получает новый array, `a` не тронут.
![[Slice от slice баги#^slice-fix-full-slice]]

Какие два способа изолировать подсрез от оригинала?
?
![[Slice от slice баги#^slice-fix-full-slice]] + ![[Slice от slice баги#^slice-fix-copy]]

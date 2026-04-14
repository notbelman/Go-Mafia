#flashcards/slice/subslicing

`s[i:j]` — включён ли `j`? Создаётся ли новый underlying array?
?
![[Subslicing нарезка#^sub-same-array]]

Чему равен cap производного слайса без третьего аргумента?
?
![[Subslicing нарезка#^sub-cap-rule]]

Может ли длина производного слайса быть больше длины базового? А cap?
?
![[Subslicing нарезка#^sub-len-vs-cap]]

Валидны ли операции `s[0:0]` и `s[:0]` для nil slice? Что с `append(s, 1)` и `for range s`?
?
![[Subslicing нарезка#^sub-nil-ops]]

Как расширить `len` слайса до его `cap` через нарезку?
?
![[Subslicing нарезка#^sub-resize]]

Поддерживает ли Go отрицательные индексы при нарезке?
?
![[Subslicing нарезка#^sub-no-negative]]

```go
s := make([]int, 4, 6)
sub := s[2:4]
```
Чему равны `len(sub)` и `cap(sub)`?
?
![[Subslicing нарезка#^sub-len-cap-example]]
![[Subslicing нарезка#^sub-cap-rule]]

Что произойдёт с `data`, если изменить `sub[0]`?
```go
data := []int{1, 2, 3, 4, 5, 0}
sub := data[1:3]  

sub[0] = 999
fmt.Println(data) 

sub = append(sub, 77) 
```
?
![[Subslicing нарезка#^sub-sharing-example]]

Что выведет этот код?
```go
data := []int{1, 2, 3, 4, 5, 0}
sub := data[1:3]
sub = append(sub, 77)
fmt.Println(data)
fmt.Println(sub)
```
?
`[1 2 3 77 5 0]` и `[2 3 77]` — append пишет в тот же array на позицию data[3], потому что cap(sub)=5 > len(sub)=2. len(data) не меняется.
![[Subslicing нарезка#^sub-sharing-why]]

Почему `s[3] = 42` паникует после `make([]int, 3, 6)`, но работает после `s = s[:cap(s)]`?
?
`s[i]` ограничен `len`. После `make` len=3, индекс 3 за границей → паника. `s[:cap(s)]` расширяет len до 6 через reslice, теперь индекс 3 в пределах len.
![[Subslicing нарезка#^sub-resize-example]]
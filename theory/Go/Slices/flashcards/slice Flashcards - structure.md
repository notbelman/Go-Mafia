#flashcards/slice/structure

Из каких полей состоит внутренняя структура slice в Go? Назови типы.
?
![[Slice структура#^slice-struct-fields]]

Сколько байт занимает slice header на 64-bit платформе? Почему именно столько?
?
![[Slice структура#^slice-header-size]]

В чём разница между `len` и `cap` у slice?
?
![[Slice структура#^slice-len-cap-meaning]]

Что означает "cap считается от текущего ptr"? Нарисуй мысленно, если взять подсрез `s[2:]`.
?
![[Slice структура#^slice-len-cap-meaning]]

Что выведет этот код?
```go
s := []int{1, 2, 3, 4, 5}
s2 := s[2:]
fmt.Println(len(s2), cap(s2))
```
?
`3 3` — `len(s2) = 5-2 = 3`, `cap(s2) = 5-2 = 3`. cap считается от нового ptr до конца underlying array.
![[Slice структура#^slice-len-cap-meaning]]

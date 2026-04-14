#flashcards/slice/len_cap

Что означает `len` слайса?
?
![[len vs cap#^lencap-len-def]]

Что означает `cap` слайса?
?
![[len vs cap#^lencap-cap-def]]

Почему `s[3]` вызывает panic, если `len(s) = 2`, даже если `cap(s) = 5`?
?
![[len vs cap#^lencap-index-bounds]]

Что выведет этот код?
```go
s := make([]int, 2, 5)
s = s[:4]
fmt.Println(len(s), cap(s))
s = s[:6]
```
?
Сначала выводит `4 5` — reslice до 4 успешен, cap остаётся 5. Затем panic: `s[:6]` превышает cap=5. Индексирование ограничено len, reslice — cap.
![[len vs cap#^lencap-index-bounds]]

В чём разница между `s[3] = 4` и `s = s[:4]` при `make([]int, 2, 5)`?
?
![[len vs cap#^lencap-index-bounds]]

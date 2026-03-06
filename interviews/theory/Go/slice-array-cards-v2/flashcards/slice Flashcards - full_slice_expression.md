#flashcards/slice/full_slice_expression

Что означает `s[i:j:k]`? Чему равны len и cap результата?
?
![[Full slice expression#^fse-syntax]]

Какой аргумент full slice expression можно опустить, а какие нельзя?
?
![[Full slice expression#^fse-omit-rules]]

Что произойдёт при компиляции `a[1::6]` и `a[1:4:]`?
?
![[Full slice expression#^fse-omit-rules]]

Почему без третьего аргумента в `s[i:j]` append может перезаписать оригинальный массив?
?
![[Full slice expression#^fse-why-needed]]

Что выведет этот код?
```go
a := []int{1, 2, 3, 4, 5}
b := a[:2]
b = append(b, 99)
fmt.Println(a)
```
?
`[1 2 99 4 5]` — `b` наследует cap=5 от `a`. append пишет 99 в позицию `a[2]`, шаря underlying array. Оригинал перезаписан.
![[Full slice expression#^fse-why-needed]]

Что выведет этот код?
```go
a := []int{1, 2, 3, 4, 5}
b := a[:2:2]
b = append(b, 99)
fmt.Println(a)
```
?
`[1 2 3 4 5]` — `b` имеет cap=2=len. append не влезает в существующий массив, создаёт новый. Оригинал не тронут.
![[Full slice expression#^fse-pattern-cap-equals-len]]

Зачем использовать паттерн `a[:n:n]`? В каком случае это критично?
?
![[Full slice expression#^fse-pattern-cap-equals-len]]

Чему равны len и cap для `a[1:4:5]` если `a = []int{1,2,3,4,5}`?
?
`len = 4-1 = 3`, `cap = 5-1 = 4`. Формула: `len = high - low`, `cap = max - low`.
![[Full slice expression#^fse-syntax]]

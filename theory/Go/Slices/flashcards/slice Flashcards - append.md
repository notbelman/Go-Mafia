#flashcards/slice/append

При каком условии append выделяет новый underlying array?
?
![[append#^append-rule-realloc]]

Почему нужно всегда писать `s = append(s, x)`, а не просто `append(s, x)`?
?
![[append#^append-assign-required]]

Каков порог роста cap в Go 1.18+ и что происходит до и после него?
?
![[append#^append-growth-summary]]

Почему амортизированная сложность append — O(1), хотя реаллокация — O(n)?
?
![[append#^append-complexity-why]]

Докажи что амортизированная сложность append — O(1). Сколько элементов копируется за N операций?
?
![[append#^append-complexity-proof]]

Когда и зачем использовать `make([]int, 0, n)` перед серией append?
?
![[append#^append-prealloc]]

Опиши формулу роста cap в Go 1.18+. Что такое `threshold` и как он используется?
?
![[append#^append-growth-formula]]

Почему append — функция, а не метод на slice? Назови все три причины.
?
![[append#^append-func-reason-header]]
![[append#^append-func-reason-unnamed]]
![[append#^append-func-reason-return]]

Почему append не может быть методом с value receiver?
?
![[append#^append-func-reason-header]]

Почему append не может быть методом вообще (не считая value receiver)?
?
![[append#^append-func-reason-unnamed]]

Что выведет этот код?
```go
s := make([]int, 2, 3)
s = append(s, 1)
s = append(s, 2)
fmt.Println(len(s), cap(s))
```
?
`4 6` (или другой удвоенный cap ≥ 4) — первый append влезает в cap=3, второй требует реаллокацию. cap < 256, поэтому удваивается: 3*2 = 6.
![[append#^append-rule-realloc]]

Что выведет этот код?
```go
s := make([]int, 0, 5)
s = append(s, 1, 2, 3)
s2 := s[:2]
s2 = append(s2, 99)
fmt.Println(s)
```
?
`[1 2 99]` — append в s2 пишет в тот же underlying array, потому что cap(s2) = 5 и места хватает. s и s2 шарят массив, s[2] перезаписан.
![[append#^append-rule-realloc]]

Что выведет этот код?
```go
a := make([]int, 0, 3)
b := append(a, 1, 2, 3)
c := append(a, 4, 5, 6)
fmt.Println(b)
fmt.Println(c)
```
?
`[4 5 6]` и `[4 5 6]` — оба append пишут в тот же underlying array (cap=3, len(a)=0). c перезаписывает то что записал b. b и c указывают на один массив.
![[append#^append-rule-realloc]]
![[append#^append-assign-required]]
+
![[append#Как append работает внутри]]

Как append записывает элементы, если в underlying array есть место?
?
Записывает в позицию `len` underlying array и увеличивает `len`. Реаллокации не происходит.
![[append#^append-mechanic]]

Что именно проверяет append перед записью?
?
`len + количество новых элементов > cap`. Если да — реаллокация, если нет — запись в существующий массив.
![[append#^append-mechanic]]

Почему `make([]int, 0, 3)` + `append(s, 1)` перезаписывает нули в underlying array?
?
Нули — это zero values инициализации, а не данные слайса. `len=0` означает «0 используемых элементов». `append` пишет в позицию `len` (arr[0]) и увеличивает `len` до 1.
![[append#^append-mechanic]]
![[Создание slice#^make-zero-values]]
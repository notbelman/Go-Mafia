#flashcards/slice/create

Какие три способа создать slice в Go?
?
![[Создание slice#^create-three]]

Чему равен cap при `arr[1:3]` если `arr` — массив длиной 5?
?
![[Создание slice#^cap-formula]]

Как вычислить cap у slice, созданного нарезкой от массива `arr[i:j]`?
?
![[Создание slice#^cap-formula]]

При `s := arr[1:3]` — куда указывает ptr слайса? Что произойдёт при изменении `s[0]`?
?
![[Создание slice#^slice-shares-array]]

Что выведет этот код?
```go
arr := [5]int{1, 2, 3, 4, 5}
s := arr[1:3]
s[0] = 99
fmt.Println(arr)
```
?
`[1 99 3 4 5]` — ptr слайса указывает на `arr[1]`, изменение через slice меняет исходный массив.
![[Создание slice#^slice-shares-array]]

Что выведет этот код?
```go
arr := [5]int{1, 2, 3, 4, 5}
s := arr[1:3]
fmt.Println(len(s), cap(s))
```
?
`2 4` — len = j - i = 3 - 1 = 2; cap = len(arr) - i = 5 - 1 = 4.
![[Создание slice#^cap-formula]]

Чем заполнен underlying array после `make([]int, 0, 3)`?
?
Zero values для типа: `[0, 0, 0]`. Элементы не доступны через слайс (`len=0`), но массив уже выделен и инициализирован.
![[Создание slice#^make-zero-values]]

Что такое zero value в Go? Приведи примеры для основных типов.
?
Значение по умолчанию при инициализации: `0` для числовых типов, `""` для string, `false` для bool, `nil` для указателей, слайсов, map и каналов.
![[Создание slice#^make-zero-values]]

Что выведет этот код?
```go
s := make([]string, 3)
fmt.Println(s[0] == "")
fmt.Println(len(s), cap(s))
```
?
`true` и `3 3` — `make([]string, 3)` создаёт слайс с len=3, cap=3, заполненный zero values для string (`""`).
![[Создание slice#^make-zero-values]]
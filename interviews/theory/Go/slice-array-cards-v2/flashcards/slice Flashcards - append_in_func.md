#flashcards/slice/append_in_func

Что именно копируется при передаче слайса в функцию? Что из этого общее?
?
![[append внутри функции — НЕ видно снаружи#^app-slice-header]]

Если `cap == len` и внутри функции делают `append` — что увидит caller?
?
![[append внутри функции — НЕ видно снаружи#^app-cap-eq-len]]

Если `cap > len` и внутри функции делают `append` — что увидит caller?
?
![[append внутри функции — НЕ видно снаружи#^app-cap-gt-len]]

Почему `copy := append(data, 5)` при `cap > len` — опасный паттерн?
?
![[append внутри функции — НЕ видно снаружи#^app-dangerous-case]]

Два способа сделать изменения через append видимыми снаружи функции?
?
![[append внутри функции — НЕ видно снаружи#^app-fix]]

Что выведет этот код?
```go
func add(s []int) {
    s = append(s, 4)
}
s := []int{1, 2, 3}
add(s)
fmt.Println(s)
```
?
`[1 2 3]` — len=cap=3, append создаёт новый array внутри функции. Caller не видит изменений.
![[append внутри функции — НЕ видно снаружи#^app-example-new-array]]

Что выведет этот код?
```go
func add(s []int) {
    s = append(s, 4)
    s[0] = 999
}
s := make([]int, 3, 10)
s[0], s[1], s[2] = 1, 2, 3
add(s)
fmt.Println(s)
```
?
`[999 2 3]` — cap>len, append пишет в тот же array. s[0]=999 меняет общие данные. Но len caller'а всё ещё 3, поэтому 4 не видна.
![[append внутри функции — НЕ видно снаружи#^app-example-same-array]]

Что выведет этот код?
```go
func bad(data []int) []int {
    return append(data, 5)
}
s := make([]int, 3, 5)
s[0], s[1], s[2] = 1, 2, 3
t := bad(s)
s2 := bad(s)
fmt.Println(t)
fmt.Println(s2)
```
?
`[1 2 3 5]` и `[1 2 3 5]` — оба append пишут на одну позицию `s[3]` в общем array, перезаписывая друг друга. Опасный паттерн: два независимых слайса шарят буфер.
![[append внутри функции — НЕ видно снаружи#^app-dangerous-case]]

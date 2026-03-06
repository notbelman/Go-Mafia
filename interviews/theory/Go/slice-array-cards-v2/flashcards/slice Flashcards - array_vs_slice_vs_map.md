#flashcards/slice/array_vs_slice_vs_map

Сколько байт передаётся при передаче slice в функцию? А map? А array [1000]int?
?
![[Массив vs Слайс vs Мапа#^pass-to-func]]

Передаёшь slice в функцию и внутри делаешь `s = append(s, x)`. Будет ли изменение len видно снаружи?
?
![[Массив vs Слайс vs Мапа#^mutation-in-func]]

Передаёшь slice в функцию и внутри меняешь `s[0] = 99`. Будет ли изменение видно снаружи?
?
![[Массив vs Слайс vs Мапа#^mutation-in-func]]

Почему array можно использовать как ключ в map, а slice — нельзя?
?
![[Массив vs Слайс vs Мапа#^comparable]]

Что такое zero value для array, slice и map?
?
![[Массив vs Слайс vs Мапа#^zero-value]]

Что произойдёт если читать из nil map? А писать в nil map?
?
![[Массив vs Слайс vs Мапа#^zero-value]]

Что выведет этот код?
```go
func modify(s []int) {
    s[0] = 99
    s = append(s, 100)
}

func main() {
    s := []int{1, 2, 3}
    modify(s)
    fmt.Println(s)
}
```
?
`[99 2 3]` — `s[0] = 99` изменяет underlying array (виден снаружи), но `append` внутри функции создаёт новый header (или расширяет существующий) локально — изменение len снаружи не видно.
![[Массив vs Слайс vs Мапа#^mutation-in-func]]

Что выведет этот код?
```go
func modify(a [3]int) {
    a[0] = 99
}

func main() {
    a := [3]int{1, 2, 3}
    modify(a)
    fmt.Println(a)
}
```
?
`[1 2 3]` — array передаётся по значению (полная копия), мутации внутри функции не видны снаружи.
![[Массив vs Слайс vs Мапа#^pass-to-func]]

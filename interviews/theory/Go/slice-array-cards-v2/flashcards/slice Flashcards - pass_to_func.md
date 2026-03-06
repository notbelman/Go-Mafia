#flashcards/slice/pass_to_func

Что копируется при передаче slice в функцию? Что НЕ копируется?
?
![[Slice передача в функцию#^slice-pass-copy-header]]

Сколько байт копируется при передаче slice в функцию?
?
![[Slice передача в функцию#^slice-pass-copy-header]]

Почему изменение элементов slice внутри функции видно снаружи?
?
![[Slice передача в функцию#^slice-pass-why-visible]]

Что выведет этот код?
```go
func modify(s []int) {
    s[0] = 999
}
func main() {
    s := []int{1, 2, 3}
    modify(s)
    fmt.Println(s)
}
```
?
`[999 2 3]` — обе копии header указывают на один underlying array, запись через индекс модифицирует его напрямую.
![[Slice передача в функцию#^slice-pass-mutate-visible]]

Почему append внутри функции НЕ виден снаружи (без реаллокации)?
?
![[Slice передача в функцию#^slice-pass-append-invisible]]

Что выведет этот код?
```go
func grow(s []int) {
    s = append(s, 100)
    fmt.Println("inside:", s)
}
func main() {
    s := make([]int, 3, 5)
    grow(s)
    fmt.Println("outside:", len(s))
}
```
?
`inside: [0 0 0 100]`, `outside: 3` — append записал 100 в underlying array (cap хватало), но len изменился только в локальной копии header. Снаружи len остался 3.
![[Slice передача в функцию#^slice-pass-append-invisible]]

Как правильно вернуть изменённый slice из функции? Два способа.
?
![[Slice передача в функцию#^slice-pass-return-pattern]]

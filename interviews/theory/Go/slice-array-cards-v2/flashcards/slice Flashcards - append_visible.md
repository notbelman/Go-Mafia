#flashcards/slice/append_visible

Какие два способа сделать append внутри функции видимым снаружи?
?
![[Как сделать append видимым#^append-visible-return]]
![[Как сделать append видимым#^append-visible-pointer]]

Как работает идиоматический способ передачи slice в функцию с append? Покажи сигнатуру.
?
![[Как сделать append видимым#^append-visible-return]]

Как работает способ через `*[]int`? Почему он работает?
?
![[Как сделать append видимым#^append-visible-pointer]]

Почему способ через `*[]int` менее идиоматичен для Go?
?
![[Как сделать append видимым#^append-visible-tradeoff]]

Что выведет этот код?
```go
func addElem(s []int) {
    s = append(s, 99)
}

func main() {
    s := []int{1, 2, 3}
    addElem(s)
    fmt.Println(s)
}
```
?
`[1 2 3]` — функция получает копию slice header. append создаёт новый header внутри функции, но оригинальный s в main не меняется. Чтобы изменения были видны, нужно вернуть slice или передать `*[]int`.
![[Как сделать append видимым#^append-visible-return]]

Что выведет этот код?
```go
func addElem(s *[]int) {
    *s = append(*s, 99)
}

func main() {
    s := []int{1, 2, 3}
    addElem(&s)
    fmt.Println(s)
}
```
?
`[1 2 3 99]` — передаётся указатель на slice header. Функция разыменовывает его и присваивает результат append напрямую, меняя оригинальный header в main.
![[Как сделать append видимым#^append-visible-pointer]]

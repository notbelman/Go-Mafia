#flashcards/slice/allocation

Каков порог размера underlying array для аллокации slice на стеке vs heap?
?
![[Slice аллокация стек и хип#^slice-alloc-threshold]]

Каков порог размера для аллокации массива (`[N]T`) на стеке? Во сколько раз он больше порога для slice?
?
![[Slice аллокация стек и хип#^slice-alloc-threshold]]

Что произойдёт с underlying array после реаллокации, если исходный slice был на стеке?
?
![[Slice аллокация стек и хип#^slice-alloc-realloc-heap]]

Что выведет (и где аллоцируется) этот код?
```go
s1 := make([]byte, 65_536)
s2 := make([]byte, 65_537)
_ = s1
_ = s2
```
?
`s1` — стек (64 КБ = порог), `s2` — heap (64 КБ + 1 > порог). Вывода нет, вопрос об аллокации.
![[Slice аллокация стек и хип#^slice-alloc-threshold]]

Как обойти порог 64 КБ и разместить большой underlying array на стеке?
?
![[Slice аллокация стек и хип#^slice-alloc-hack-stack]]

Почему трюк `var arr [1_000_000]byte; s := arr[:]` работает? Какое ограничение у массива на стеке?
?
![[Slice аллокация стек и хип#^slice-alloc-hack-why]]

Почему сигнатура `io.Reader.Read(p []byte)` экономичнее чем `Read() []byte`?
?
![[Slice аллокация стек и хип#^slice-alloc-ioreader-why]]

Почему `Read() []byte` вынуждает аллоцировать буфер в heap?
?
![[Slice аллокация стек и хип#^slice-alloc-ioreader-sig]]

Что выведет этот код с точки зрения аллокации?
```go
func main() {
    s := make([]byte, 0, 3)
    s = append(s, 1, 2, 3)
    s = append(s, 4) // cap overflow
    _ = s
}
```
?
Первый `make` — стек (3 байта << 64 КБ). После второго `append` (cap overflow) — реаллокация, новый array в **heap** независимо от исходного размера.
![[Slice аллокация стек и хип#^slice-alloc-realloc-heap]]

#flashcards/functions/defer_mechanics

К чему привязан defer — к функции или к блоку (scope)?
?
![[Defer механика и порядок#^dm-scope]]
<!--SR:!2026-02-28,4,270-->

В каком порядке выполняются несколько defer в одной функции?
?
![[Defer механика и порядок#^dm-lifo]]
<!--SR:!2026-02-27,3,250-->

Что выведет этот код?
```go
func main() {
    defer fmt.Println("A")
    defer fmt.Println("B")
    defer fmt.Println("C")
}
```
?
`C`, `B`, `A` — LIFO (Last In, First Out). Последний отложенный defer выполняется первым.
![[Defer механика и порядок#^dm-lifo-example]]
<!--SR:!2026-02-27,3,250-->

Какие две runtime-функции стоят за defer под капотом?
?
![[Defer механика и порядок#^dm-runtime]]
<!--SR:!2026-02-27,3,250-->

Что выведет этот код?
```go
func example() {
    for _, p := range []string{"a", "b", "c"} {
        f, _ := os.Open(p)
        defer f.Close()
        process(f)
    }
}
```
?
Все три `f.Close()` будут вызваны только при выходе из `example()`, а не после каждой итерации. Пока функция работает — открыты все 3 файла одновременно. Это утечка файловых дескрипторов.
![[Defer механика и порядок#^dm-not-block]]
<!--SR:!2026-02-27,3,250-->

Чем defer в Go отличается от RAII в C++?
?
![[Defer механика и порядок#^dm-not-block]]
<!--SR:!2026-02-27,3,250-->

Что произойдёт с defer внутри `if false { defer f() }`?
?
![[Defer механика и порядок#^dm-unreachable]]
![[Defer механика и порядок#^dm-unreachable-example]]
<!--SR:!2026-02-27,3,250-->

Почему лучше объединять несколько defer в один через анонимную функцию?
?
![[Defer механика и порядок#^dm-combine-example]]
<!--SR:!2026-02-27,3,250-->

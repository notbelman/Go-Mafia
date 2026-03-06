#flashcards/channels-other/directional

Запиши синтаксис трёх видов направленности каналов в Go.
?
![[Направленные каналы (Directional Channels)#^directional-syntax]]

Направленность канала — это ограничение компилятора или рантайма? Что под капотом?
?
![[Направленные каналы (Directional Channels)#^directional-compiler-only]]

Можно ли расширить `chan<- int` обратно до `chan int`? Почему?
?
![[Направленные каналы (Directional Channels)#^directional-narrowing]]
![[Направленные каналы (Directional Channels)#^directional-one-way]]

Зачем нужны направленные каналы? Какую проблему они решают?
?
![[Направленные каналы (Directional Channels)#^directional-why]]

Что выведет этот код?
```go
func producer(out chan<- int) {
    out <- 42
}
func main() {
    ch := make(chan int)
    go producer(ch)
    fmt.Println(<-ch)
}
```
?
`42` — `ch` автоматически сужается до `chan<- int` при передаче в `producer`. Сужение типа происходит неявно, под капотом тот же `hchan`.
![[Направленные каналы (Directional Channels)#^directional-compiler-only]]
![[Направленные каналы (Directional Channels)#^directional-example]]

Что произойдёт при компиляции?
```go
func producer(out chan<- int) {
    val := <-out
    _ = val
}
```
?
`compile error: cannot receive from send-only channel` — направленность проверяется компилятором, не рантаймом.
![[Направленные каналы (Directional Channels)#^directional-compiler-only]]

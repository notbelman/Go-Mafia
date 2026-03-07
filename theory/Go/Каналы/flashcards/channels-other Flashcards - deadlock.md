#flashcards/channels-other/deadlock

Назови 5 типичных сценариев deadlock с каналами.
?
![[Deadlock в каналах#^deadlock-cases]]

Что выведет этот код и почему?
```go
func main() {
    ch := make(chan int)
    ch <- 1
}
```
?
`fatal error: all goroutines are asleep - deadlock!` — небуферизированный канал требует встречного receiver. В одной горутине send и receive встретиться не могут.
![[Deadlock в каналах#^deadlock-unbuffered-single]]

Что выведет этот код?
```go
func main() {
    ch := make(chan int, 1)
    ch <- 1
    ch <- 2
    fmt.Println(<-ch)
}
```
?
`fatal error: all goroutines are asleep - deadlock!` — буфер размером 1 заполнен после первого send. Второй `ch <- 2` блокирует, receiver нет → deadlock.
![[Deadlock в каналах#^deadlock-full-buffer]]

Что происходит при чтении или записи в `nil` канал (не panic)?
?
![[Deadlock в каналах#^deadlock-nil-channel]]

Что делает `select {}`?
?
![[Deadlock в каналах#^deadlock-empty-select]]

Что выведет этот код?
```go
func main() {
    var ch chan int
    fmt.Println("before")
    <-ch
    fmt.Println("after")
}
```
?
`before` затем `fatal error: all goroutines are asleep - deadlock!` — nil канал блокирует навсегда, это не panic.
![[Deadlock в каналах#^deadlock-nil-channel]]

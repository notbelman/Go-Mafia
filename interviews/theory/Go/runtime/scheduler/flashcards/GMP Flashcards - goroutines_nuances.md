#flashcards/GMP/goroutines_nuances

Что именно делает инструкция `go`? Запускает ли она горутину немедленно?
?
![[Нюансы горутин#^gn-go-not-immediate]]

Что произойдёт с замыканием переменной цикла в Go до версии 1.22?
?
![[Нюансы горутин#^gn-closure-pre122]]

Что изменилось в Go 1.22 в отношении замыкания переменной цикла?
?
![[Нюансы горутин#^gn-closure-post122]]

Что делает runtime.Goexit() вызванный из main горутины? Что происходит с другими горутинами?
?
![[Нюансы горутин#^gn-goexit-mechanics]]

Чем завершается программа после runtime.Goexit() в main? Какой exit code?
?
![[Нюансы горутин#^gn-goexit-conclusion]]

Когда завершается main горутина — что происходит с defer других горутин?
?
![[Нюансы горутин#^gn-main-exit]]

Для чего используется runtime.LockOSThread()? Приведи конкретные кейсы.
?
![[Нюансы горутин#^gn-lock-os-thread]]

Что выведет этот код?
```go
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i)
    }()
}
time.Sleep(time.Second)
```
?
До Go 1.22: скорее всего `3 3 3` — все горутины захватывают одну переменную `i`, которая к моменту запуска уже равна 3. С Go 1.22: `0 1 2` (в произвольном порядке) — каждая итерация создаёт новую переменную.
![[Нюансы горутин#^gn-closure-pre122]]

Что выведет этот код?
```go
func main() {
    go func() {
        time.Sleep(100 * time.Millisecond)
        fmt.Println("goroutine done")
    }()
    fmt.Println("main done")
}
```
?
`main done` — и только. Программа завершается вместе с main горутиной. Горутина не успевает выполниться, её defer не вызываются.
![[Нюансы горутин#^gn-main-exit]]

Что выведет этот код?
```go
func main() {
    go func() {
        fmt.Println("goroutine")
    }()
    runtime.Goexit()
    fmt.Println("unreachable")
}
```
?
Выведет `goroutine`, затем программа крашнется с `fatal error: no goroutines (main called runtime.Goexit) - deadlock!`, exit code 2. main горутина завершена Goexit, spawn-горутина отработала, runtime обнаружил deadlock.
![[Нюансы горутин#^gn-goexit-mechanics]]

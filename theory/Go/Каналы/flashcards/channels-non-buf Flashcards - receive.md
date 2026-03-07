#flashcards/channels-non-buf/receive

Что произойдёт при receive из nil-канала?
?
![[Receive (-ch)#^recv-nil]]

Что возвращает receive из закрытого канала? Блокирует ли он горутину?
?
![[Receive (-ch)#^recv-closed]]

Опиши механизм "Direct Copy" при receive — пошагово что происходит когда sender уже ждёт в sendq.
?
![[Receive (-ch)#^recv-direct-copy]]

При Direct Copy receive — кто вызывает goready и для какой горутины?
?
![[Receive (-ch)#^recv-direct-copy]]

Опиши механизм receive когда sendq пуст — что создаётся, куда встаём, что происходит дальше?
?
![[Receive (-ch)#^recv-sleep]]

Что такое `sudog` при receive и что хранится в поле `elem`?
?
![[Receive (-ch)#^recv-sleep]]

Кто и когда записывает данные в стек заблокированного получателя?
?
![[Receive (-ch)#^recv-sleep]]

В каком порядке receive проверяет условия перед тем как принять решение?
?
![[Receive (-ch)#^recv-order]]

Что выведет этот код?
```go
var ch chan int
go func() {
    fmt.Println(<-ch)
}()
time.Sleep(time.Second)
fmt.Println("main done")
```
?
`main done` — горутина заблокирована навечно на receive из nil-канала, но main не ждёт её. Программа завершится после `main done`, горутина утечёт.
![[Receive (-ch)#^recv-nil]]

Что выведет этот код?
```go
ch := make(chan int)
close(ch)
v, ok := <-ch
fmt.Println(v, ok)
```
?
`0 false` — receive из закрытого небуферизированного канала немедленно возвращает zero value (0 для int) и false без блокировки.
![[Receive (-ch)#^recv-closed]]

Что выведет этот код?
```go
ch := make(chan int)
go func() { ch <- 42 }()
time.Sleep(time.Millisecond)
x := <-ch
fmt.Println(x)
```
?
`42` — горутина-отправитель заблокировалась в sendq. Когда main делает receive, срабатывает Direct Copy: значение копируется из стека горутины напрямую в x, горутина пробуждается через goready.
![[Receive (-ch)#^recv-direct-copy]]

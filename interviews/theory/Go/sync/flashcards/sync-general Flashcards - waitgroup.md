#flashcards/sync-general/waitgroup

Для чего используется sync.WaitGroup?
?
![[sync.WaitGroup#^wg-definition]]

Опиши API WaitGroup — три метода и их правила.
?
![[sync.WaitGroup#^wg-api-table]]

Почему Add(1) нужно вызывать ДО запуска горутины, а не внутри неё?
?
![[sync.WaitGroup#^wg-add-before-goroutine]]

Что произойдёт если Done() вызвать больше раз, чем Add()?
?
![[sync.WaitGroup#^wg-negative-panic]]

Опиши внутреннюю структуру WaitGroup. Как в одном uint64 упакованы counter и waiters?
?
![[sync.WaitGroup#^wg-struct]]
![[sync.WaitGroup#^wg-state-packing]]

Что выведет этот код?
```go
var wg sync.WaitGroup
for i := 0; i < 3; i++ {
    go func() {
        wg.Add(1)
        defer wg.Done()
        time.Sleep(time.Millisecond)
    }()
}
wg.Wait()
fmt.Println("done")
```
?
Код содержит гонку — `wg.Wait()` может вернуться до того как все горутины успели вызвать `Add(1)`. Программа может напечатать `done` и завершиться, не дождавшись горутин. `Add(1)` нужно вызывать ДО `go func()`.
![[sync.WaitGroup#^wg-add-before-goroutine]]

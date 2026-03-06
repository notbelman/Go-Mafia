#flashcards/sync-primitive-supple/deadlock

Что такое deadlock? Что должно произойти чтобы он возник?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-definition]]

При каком условии runtime Go обнаруживает deadlock? Почему в реальных сервисах deadlock часто не виден?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-runtime-detection]]

Что выведет этот код?
```go
func main() {
    go func() {
        for { time.Sleep(time.Second) }
    }()
    var mu sync.Mutex
    mu.Lock()
    mu.Lock()
}
```
?
Программа зависнет навсегда без паники и без сообщения "all goroutines are asleep" — одна живая горутина мешает runtime обнаружить deadlock.
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-hidden]]

Почему несогласованный порядок захвата мьютексов приводит к deadlock? Покажи механизм.
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-lock-order]]

Как решить проблему несогласованного порядка захвата мьютексов?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-lock-order-solution]]

Почему в Go можно получить deadlock одним мьютексом? Что происходит при повторном Lock?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-not-reentrant]]

Что выведет этот код?
```go
func main() {
    var mu sync.Mutex
    mu.Lock()
    mu.Lock()
    fmt.Println("done")
}
```
?
`fatal error: all goroutines are asleep - deadlock!` — Go мьютекс не reentrant, не хранит ID владельца. Второй Lock блокируется навечно, а т.к. горутина одна — runtime видит deadlock.
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-not-reentrant]]

Можно ли разлочить мьютекс из другой горутины в Go?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-unlock-other-goroutine]]

Что произойдёт при Unlock незалоченного мьютекса?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-unlock-panic]]

Для deadlock концептуально нужно 2 мьютекса. Почему в Go достаточно одного?
?
![[interviews/theory/Go/Concurrency Glossary/Deadlock#^deadlock-one-mutex]]

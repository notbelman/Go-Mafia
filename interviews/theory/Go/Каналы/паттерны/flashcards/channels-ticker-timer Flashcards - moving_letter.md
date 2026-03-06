#flashcards/channels-patterns/moving_letter

Что такое Moving Letter? Когда применяется?
?
![[Moving Letter#^ml-idea]]

Что выведет и произойдёт с этим кодом?
```go
func query(addrs []string) string {
    resultCh := make(chan string) // небуферизированный!
    for _, addr := range addrs {
        go func(a string) {
            result := fetchFrom(a)
            select {
            case resultCh <- result:
            default:
            }
        }(addr)
    }
    return <-resultCh
}
```
?
**Deadlock** (или непредсказуемый результат): если все горутины успеют завершиться до того как `<-resultCh` будет готов читать, все уйдут в `default` и читатель заблокируется навсегда. Нужен буфер.
![[Moving Letter#^ml-buf-required]]

Почему размер буфера `len(addrs)`, а не 1? Что хуже — буфер 1 или len(addrs)?
?
![[Moving Letter#^ml-buf-1-vs-n]]

Опиши трейдоффы Moving Letter.
?
![[Moving Letter#^ml-tradeoff]]

В чём принципиальное отличие Moving Letter от Single Flight?
?
![[Moving Letter#^ml-idea]]
![[Single Flight#^sf-problem]]

#flashcards/channels-extra/nonblocking

Почему `if len(ch) > 0 { <-ch }` — race condition?
?
![[WORK-BASE/interviews/theory/Rust/Какие-то вопросы/Асинк/Неблокирующие операции#^nb-len-race]]

Что делает `select` + `default` атомарным? Что именно он гарантирует?
?
![[WORK-BASE/interviews/theory/Rust/Какие-то вопросы/Асинк/Неблокирующие операции#^nb-select-default-atomic]]

Напиши шаблоны неблокирующего чтения и неблокирующей записи через select.
?
![[WORK-BASE/interviews/theory/Rust/Какие-то вопросы/Асинк/Неблокирующие операции#Правильный подход — select + default]]

Паттерн first-response-wins: 5 горутин пишут в канал, читатель берёт первый результат. Почему используется `make(chan int, 1)` а не `make(chan int)`?
?
![[WORK-BASE/interviews/theory/Rust/Какие-то вопросы/Асинк/Неблокирующие операции#^nb-frw-buf1-reason]]

Что гарантирует буфер=1 в паттерне first-response-wins?
?
![[WORK-BASE/interviews/theory/Rust/Какие-то вопросы/Асинк/Неблокирующие операции#^nb-frw-buf1-guarantee]]

Что выведет этот код (примерно — порядок горутин недетерминирован)?
```go
ch := make(chan int, 1)
for i := 0; i < 3; i++ {
    i := i
    go func() {
        select {
        case ch <- i:
        default:
        }
    }()
}
time.Sleep(10 * time.Millisecond)
fmt.Println(<-ch)
```
?
Одно из значений `0`, `1` или `2` — первая горутина, успевшая записать, кладёт значение в буфер=1. Остальные идут в `default` и завершаются без блокировки. Какая именно победит — недетерминировано.
![[WORK-BASE/interviews/theory/Rust/Какие-то вопросы/Асинк/Неблокирующие операции#^nb-frw-buf1-guarantee]]

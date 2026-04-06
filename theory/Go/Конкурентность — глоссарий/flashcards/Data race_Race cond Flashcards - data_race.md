#flashcards/dr_and_rc/data_race

Что такое data race? Назови все необходимые условия.
?
![[Data Race#^dr-definition]]

Почему data race приводит к Undefined Behavior, а не просто к неверному результату?
?
![[Data Race#^dr-ub]]

Почему data race может дать непредсказуемый результат даже если логика кажется простой?
?
![[Data Race#^dr-ub-reasons]]

Назови 4 способа исправить data race. Когда что использовать?
?
![[Data Race#^dr-fixes]]

Что выведет этот код?
```go
func main() {
    var n int
    go func() { n++ }()
    fmt.Println(n)
}
```
?
Результат непредсказуем (UB). Скорее всего `0`, но это data race: горутина пишет `n`, main читает `n` без happens-before между ними.
![[Data Race#^dr-example]]

Что выведет этот код?
```go
func main() {
    var wg sync.WaitGroup
    n := 0
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            n++
        }()
    }
    wg.Wait()
    fmt.Println(n)
}
```
?
Результат непредсказуем — data race. `n++` это read-modify-write, не атомарная операция. Результат может быть меньше 1000. Нужен `atomic.AddInt64` или mutex.
![[Data Race#^dr-fixes]]

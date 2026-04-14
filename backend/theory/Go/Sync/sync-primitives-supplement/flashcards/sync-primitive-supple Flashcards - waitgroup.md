#flashcards/sync-primitive-supple/waitgroup

Что произойдёт если передать WaitGroup по значению в горутину?
?
![[WaitGroup_копирование_и_нюансы#^wg-copy-bug]]

Что выведет этот код?
```go
func worker(wg sync.WaitGroup) {
    defer wg.Done()
    time.Sleep(10 * time.Millisecond)
}

func main() {
    var wg sync.WaitGroup
    wg.Add(1)
    go worker(wg)
    wg.Wait()
    fmt.Println("done")
}
```
?
`fatal error: all goroutines are asleep - deadlock!` — worker получил копию WaitGroup. Done вызывается на копии, оригинальный счётчик остаётся 1, Wait() никогда не вернётся.
![[WaitGroup_копирование_и_нюансы#^wg-copy-internals]]

Какие типы из пакета sync нельзя копировать после первого использования?
?
![[WaitGroup_копирование_и_нюансы#^wg-sync-no-copy-list]]

Почему Add(n) перед циклом лучше чем n раз Add(1) внутри цикла?
?
![[WaitGroup_копирование_и_нюансы#^wg-add-n-reason]]

Во сколько раз атомарный инкремент медленнее обычного?
?
![[WaitGroup_копирование_и_нюансы#^wg-add-optimization]]

Что произойдёт при вызове Wait() если счётчик WaitGroup равен 0?
?
![[WaitGroup_копирование_и_нюансы#^wg-wait-zero]]

Что произойдёт при вызове Add(-10) если текущий счётчик WaitGroup равен 5?
?
![[WaitGroup_копирование_и_нюансы#^wg-negative-panic]]

Где аллоцируется WaitGroup — в heap или stack — если на неё утекает указатель?
?
![[WaitGroup_копирование_и_нюансы#^wg-escape-analysis]]

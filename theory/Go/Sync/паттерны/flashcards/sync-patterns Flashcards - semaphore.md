#flashcards/sync-patterns/semaphore

Что такое семафор? Опиши операции Acquire и Release.
?
![[Семафор#^semaphore-operations]]

Откуда названия P и V в операциях семафора?
?
![[Семафор#^semaphore-pv-origin]]

Кто и когда придумал семафор?
?
![[Семафор#^semaphore-pv-origin]]

Как реализовать семафор на Go через буферизированный канал?
?
![[Семафор#^semaphore-buffered-chan]]

Чем бинарный семафор отличается от мьютекса?
?
![[Семафор#^semaphore-binary]]
![[Семафор#^semaphore-no-owner]]

Для чего используется считающий семафор? Приведи пример.
?
![[Семафор#^semaphore-counting]]

Что такое взвешенный семафор в Go? Пакет и когда использовать?
?
![[Семафор#^semaphore-weighted]]

В чём ключевое отличие семафора от мьютекса по понятию владельца?
?
![[Семафор#^semaphore-no-owner]]

Как используются семафоры внутри Go runtime? Какие примитивы строятся на них?
?
![[Семафор#^semaphore-go-runtime-internal]]

Что выведет этот код?
```go
sem := make(chan struct{}, 2)

var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        sem <- struct{}{}
        fmt.Printf("working %d\n", id)
        time.Sleep(50 * time.Millisecond)
        <-sem
    }(i)
}
wg.Wait()
```
?
Выводит 5 строк "working N" в неопределённом порядке, но в любой момент времени работают максимум 2 горутины. Буферизированный канал размером 2 — это семафор на 2 слота. Следующие горутины блокируются пока кто-то не освободит слот.
![[Семафор#^semaphore-buffered-chan]]
![[Семафор#^semaphore-counting]]

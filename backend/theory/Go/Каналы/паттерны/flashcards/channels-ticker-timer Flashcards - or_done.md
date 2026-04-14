#flashcards/channels-patterns/or_done

Какую проблему решает or-done channel? Что без него?
?
![[Or-done channel#^ordone-problem]]

Как выглядит использование orDone в коде потребителя?
?
![[Or-done channel#^ordone-solution]]

Опиши реализацию orDone. Почему там двойной select?
?
![[Or-done channel#^ordone-impl]]
![[Or-done channel#^ordone-double-select]]

Три сценария завершения orDone — что происходит в каждом?
?
![[Or-done channel#^ordone-how]]

Что выведет этот код?
```go
done := make(chan struct{})
ch := make(chan int)
go func() { ch <- 1; ch <- 2; ch <- 3 }()
go func() { time.Sleep(50*time.Millisecond); close(done) }()
for v := range orDone(done, ch) {
    fmt.Println(v)
    time.Sleep(100 * time.Millisecond)
}
fmt.Println("stopped")
```
?
Напечатает `1` и `stopped` — после 1-го значения горутина спит 100ms, за это время done закрывается (50ms), следующая итерация orDone обнаружит done и завершит вывод. Точный результат зависит от планировщика, но done имеет приоритет.
![[Or-done channel#^ordone-double-select]]

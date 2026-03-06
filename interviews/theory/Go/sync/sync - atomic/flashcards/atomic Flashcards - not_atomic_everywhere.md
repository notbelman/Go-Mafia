#flashcards/atomic/not_atomic_everywhere

Почему нельзя защитить два связанных поля структуры двумя atomic операциями?
?
![[Почему НЕ atomic везде#^not-atomic-multi-var-why]]

Что не так с этим кодом? Как починить?
```go
s.count.Add(1)
s.sum.Add(val)
```
?
![[Почему НЕ atomic везде#^not-atomic-multi-var]]

Почему паттерн "read-then-write" нельзя реализовать через atomic без CAS?
?
![[Почему НЕ atomic везде#^not-atomic-read-write]]

Что не так с этим кодом?
```go
if counter.Load() < 100 {
    counter.Add(1)
}
```
Как это правильно исправить?
?
![[Почему НЕ atomic везде#^not-atomic-read-write]]
![[Почему НЕ atomic везде#^not-atomic-cas-loop-needed]]

Что такое ABA problem в контексте lock-free кода?
?
![[Почему НЕ atomic везде#^not-atomic-aba]]

Почему memory ordering особенно опасен на ARM и PowerPC?
?
![[Почему НЕ atomic везде#^not-atomic-arm-ordering]]

Сколько возможных interleavings при 4 потоках и 10 операциях? Сколько покрывают обычные тесты?
?
![[Почему НЕ atomic везде#^not-atomic-interleavings]]
![[Почему НЕ atomic везде#^not-atomic-tests-coverage]]

Почему баги в lock-free коде невоспроизводимы?
?
![[Почему НЕ atomic везде#^not-atomic-bugs-unreproducible]]

Перечисли 3 основные проблемы из-за которых lock-free код называют "адом".
?
![[Почему НЕ atomic везде#^not-atomic-aba]]
![[Почему НЕ atomic везде#^not-atomic-interleavings]]
![[Почему НЕ atomic везде#^not-atomic-bugs-unreproducible]]

Что выведет этот код? Есть ли гонка данных?
```go
var counter atomic.Int64

go func() {
    if counter.Load() < 100 {
        counter.Add(1)
    }
}()
go func() {
    if counter.Load() < 100 {
        counter.Add(1)
    }
}()
time.Sleep(time.Millisecond)
fmt.Println(counter.Load())
```
?
Может вывести `1` или `2`. Нет data race (все операции atomic), но есть TOCTOU — между Load и Add другая горутина могла тоже пройти проверку. Правильное решение — CAS loop.
![[Почему НЕ atomic везде#^not-atomic-read-write]]

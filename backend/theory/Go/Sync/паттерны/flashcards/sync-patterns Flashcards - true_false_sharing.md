#flashcards/sync-patterns/true_false_sharing

Чем true sharing отличается от false sharing? В чём разница в причинах contention?
?
![[True_sharing_и_False_sharing#^true-sharing-def]]
![[True_sharing_и_False_sharing#^false-sharing-def]]

Какой размер кэш-линии типичен для современных CPU?
?
![[True_sharing_и_False_sharing#^cache-line-size]]

Что такое MESI протокол и что происходит при записи в кэш-линию?
?
![[True_sharing_и_False_sharing#^mesi-invalidation]]

Почему true sharing contention неизбежен, а false sharing — нет?
?
![[True_sharing_и_False_sharing#^true-sharing-def]]
![[True_sharing_и_False_sharing#^false-sharing-def]]

Как смягчить true sharing? Назови два подхода.
?
![[True_sharing_и_False_sharing#^true-sharing-mitigation]]

На сколько может деградировать performance при false sharing?
?
![[True_sharing_и_False_sharing#^false-sharing-perf-impact]]

Как исправить false sharing через padding? Покажи пример с конкретными числами.
?
![[True_sharing_и_False_sharing#^false-sharing-padding]]

Почему структура P в Go runtime выровнена по кэш-линиям?
?
![[True_sharing_и_False_sharing#^go-runtime-p-alignment]]

Какая утилита Linux показывает именно false sharing (cache lines с contention)?
?
![[True_sharing_и_False_sharing#^diag-perf-c2c]]

Как использовать бенчмарк для диагностики false sharing? Какой порог разницы подозрителен?
?
![[True_sharing_и_False_sharing#^diag-benchmark]]

Что выведет этот код при запуске на многоядерной машине — быстро или медленно?
```go
type Counters struct {
    a int64
    b int64
}

var c Counters
var wg sync.WaitGroup
wg.Add(2)
go func() {
    defer wg.Done()
    for i := 0; i < 10_000_000; i++ { c.a++ }
}()
go func() {
    defer wg.Done()
    for i := 0; i < 10_000_000; i++ { c.b++ }
}()
wg.Wait()
fmt.Println(c.a, c.b)
```
?
`10000000 10000000` — значения корректны, но производительность деградирована из-за false sharing: `a` и `b` в одной кэш-линии, оба ядра постоянно инвалидируют её у соседа. Фикс: добавить `_ [56]byte` между полями.
![[True_sharing_и_False_sharing#^false-sharing-def]]
![[True_sharing_и_False_sharing#^false-sharing-padding]]

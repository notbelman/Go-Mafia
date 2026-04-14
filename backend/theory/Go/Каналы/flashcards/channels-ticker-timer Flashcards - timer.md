#flashcards/ticker-timer/timer

Чем Timer отличается от Ticker по количеству срабатываний?
?
![[Timer#^ticker-vs-timer]]

Что происходит с каналом `.C` после того как Timer сработал?
?
![[Timer#^timer-after-fire]]

В чём разница между `time.NewTimer` и `context.WithTimeout` для реализации таймаута?
?
![[Timer#^timer-vs-context]]

Опиши паттерн таймаута запроса через Timer: структура и почему нет busy waiting.
?
![[Timer#^timer-timeout-pattern]]

Сравни Ticker и Timer по четырём параметрам: количество срабатываний, назначение, аналогия, пример.
?
![[Timer#^ticker-vs-timer]]

Что выведет этот код?
```go
timer := time.NewTimer(50 * time.Millisecond)
select {
case <-timer.C:
    fmt.Println("timeout")
case <-timer.C:
    fmt.Println("second read")
}
```
?
`timeout` — первый `case` сработает через 50ms. Второй `case <-timer.C` никогда не выберется, потому что после срабатывания Timer больше не пишет в канал — канал пуст.
![[Timer#^timer-after-fire]]

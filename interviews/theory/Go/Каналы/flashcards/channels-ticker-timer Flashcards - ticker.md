#flashcards/ticker-timer/ticker

Что возвращает `time.NewTicker(d)` и что приходит в канал?
?
![[Ticker#^ticker-what]]

Почему в паттерне с Ticker используют select, а не sleep?
?
![[Ticker#^ticker-select-pattern]]

Какой размер буфера у канала `.C` внутри Ticker и почему именно такой?
?
![[Ticker#^ticker-buf1]]

Что произойдёт с горутиной Ticker'а если никто не читает из `.C`?
?
![[Ticker#^ticker-buf1]]

Что выведет этот код?
```go
ticker := time.NewTicker(100 * time.Millisecond)
defer ticker.Stop()
count := 0
for t := range ticker.C {
    count++
    fmt.Println(count, t.UnixMilli()%1000)
    if count == 3 {
        return
    }
}
```
?
Выведет 3 строки вида `1 <ms>`, `2 <ms>`, `3 <ms>` с разницей ~100ms между ними. После `return` defer вызовет `Stop()`, горутина Ticker'а завершится. Если бы `Stop()` не было — горутина продолжала бы писать в канал.
![[Ticker#^ticker-internals]]

Перечисли 3-4 типичных use case для Ticker.
?
![[Ticker#^ticker-usecases]]

#flashcards/ticker-timer/leaks_gc

Почему Ticker без Stop() до Go 1.23 приводил к goroutine leak?
?
![[Ticker и Timer утечки и GC#^leak-ticker-pre123]]

Почему Timer до Go 1.23 **не** вызывал goroutine leak даже без Stop()?
?
![[Ticker и Timer утечки и GC#^leak-timer-pre123]]

Что изменилось в Go 1.23 в отношении необходимости вызывать Stop()?
?
![[Ticker и Timer утечки и GC#^gc123-stop-not-required]]

Через какой механизм Go 1.23 автоматически очищает Ticker/Timer без Stop()?
?
![[Ticker и Timer утечки и GC#^gc123-finalizers]]

Почему Stop() всё ещё считается хорошей практикой даже в Go 1.23+?
?
![[Ticker и Timer утечки и GC#^gc123-stop-good-practice]]

Что происходило при вызове `ticker.Reset(d)` до Go 1.23 если тик уже накопился в буфере?
?
![[Ticker и Timer утечки и GC#^reset-pre123]]

Как изменилось поведение `Reset` в Go 1.23 относительно буфера канала?
?
![[Ticker и Timer утечки и GC#^reset-post123]]

Что выведет этот код (Go до 1.23)?
```go
ticker := time.NewTicker(100 * time.Millisecond)
time.Sleep(200 * time.Millisecond) // тик уже в канале
ticker.Reset(500 * time.Millisecond)
start := time.Now()
<-ticker.C
fmt.Println(time.Since(start) < 50*time.Millisecond)
```
?
`true` — до Go 1.23 `Reset` не очищал буфер. Тик, накопившийся за 200ms, всё ещё в канале и прочитается мгновенно (< 50ms). В Go 1.23+ `Reset` сбрасывает буфер, поэтому было бы `false` (ожидание ~500ms).
![[Ticker и Timer утечки и GC#^reset-pre123]]

Заполни таблицу: поведение Ticker/Timer до и после Go 1.23 по 4 параметрам.
?
![[Ticker и Timer утечки и GC#^gc123-summary]]

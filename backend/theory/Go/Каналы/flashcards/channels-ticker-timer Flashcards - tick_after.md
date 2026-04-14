#flashcards/ticker-timer/tick_after

Что возвращают `time.Tick` и `time.After` в отличие от `NewTicker`/`NewTimer`?
?
![[time Tick и time After#^tick-after-no-stop]]

Почему `time.After(5s)` внутри select-цикла приводит к тому, что таймаут **никогда** не наступит?
?
![[time Tick и time After#^tick-after-loop-trap]]

Опиши пошагово что происходит в каждой итерации цикла с `time.After` внутри select.
?
![[time Tick и time After#^tick-after-loop-trap]]

Как правильно переписать select-цикл с таймером и тикером чтобы таймаут сработал через 5 секунд?
?
![[time Tick и time After#^tick-after-loop-fix]]

В каких трёх ситуациях использование `time.After` и `time.Tick` безопасно?
?
![[time Tick и time After#^tick-after-safe-after]] + ![[time Tick и time After#^tick-after-safe-tick]] + ![[time Tick и time After#^tick-after-safe-tests]]

Что выведет этот код?
```go
for i := 0; i < 3; i++ {
    select {
    case <-time.After(10 * time.Second):
        fmt.Println("timeout")
        return
    case <-time.Tick(100 * time.Millisecond):
        fmt.Println("tick", i)
    }
}
fmt.Println("done")
```
?
Выведет `tick 0`, `tick 1`, `tick 2`, затем `done`. Таймаут в 10 секунд **не сработает** — на каждой итерации создаётся новый `time.After(10s)`, старый выбрасывается. Цикл завершится через ~300ms по условию `i < 3`.
![[time Tick и time After#^tick-after-loop-trap]]

Что выведет этот код?
```go
timeout := time.After(200 * time.Millisecond)
ticker := time.Tick(100 * time.Millisecond)
for {
    select {
    case <-timeout:
        fmt.Println("done")
        return
    case <-ticker:
        fmt.Println("tick")
    }
}
```
?
Выведет `tick`, `tick`, затем `done`. `time.After` и `time.Tick` создаются **до** цикла один раз — таймаут корректно сработает через ~200ms после двух тиков по 100ms.
![[time Tick и time After#^tick-after-loop-fix]]

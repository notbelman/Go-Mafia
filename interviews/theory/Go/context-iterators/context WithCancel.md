- cancel() закрывает внутренний канал → Done() разблокируется у этого контекста и всех дочерних ^cancel-closes-channel
- cancel() безопасно вызывать несколько раз (внутри проверка: канал закрывается ровно один раз) ^cancel-idempotent
- антипаттерн: передавать cancel куда-то в другое место → спагетти-код. Вызывать там же где создали ^cancel-ownership

---

## Базовый паттерн

```go
ctx, cancel := context.WithCancel(parent)
defer cancel()  // ВСЕГДА

// запускаем 10 горутин получить погоду из разных источников
// как только первый ответил — отменяем остальные
for i := 0; i < 10; i++ {
    go fetchWeather(ctx, sources[i], resultCh)
}

result := <-resultCh
cancel()  // остальные горутины увидят ctx.Done() и остановятся
```

^cancel-fan-out-pattern

## Правильно слушать отмену: select, не последовательно

```go
// ПЛОХО: сначала читаем данные, потом проверяем отмену
// если dataCh заблокирован — отмена не дойдёт
data := <-dataCh
if ctx.Err() != nil { return }

// ХОРОШО: слушаем оба канала одновременно
select {
case data := <-dataCh:
    process(data)
case <-ctx.Done():
    return
}
```

^cancel-select-pattern

## cancel() — вызывать там же где создали

```go
// ПЛОХО: передаём cancel в другую функцию
ctx, cancel := context.WithCancel(parent)
go worker(ctx, cancel)  // кто отменяет? откуда? → спагетти

// ХОРОШО: контроль остаётся у создателя
ctx, cancel := context.WithCancel(parent)
defer cancel()
go worker(ctx)  // worker только слушает ctx.Done()
```

^cancel-ownership-example

## WithCancelCause (Go 1.20+)

```go
ctx, cancel := context.WithCancelCause(parent)

// из разных мест — разные причины
cancel(errors.New("user requested stop"))
cancel(errors.New("другая причина"))  // игнорируется! первый cancel побеждает

fmt.Println(ctx.Err())          // context canceled
fmt.Println(context.Cause(ctx)) // "user requested stop"
```

^cancel-cause-first-wins

Первый вызов cancel фиксирует причину. Повторные вызовы с другой ошибкой — ничего не меняют (канал уже закрыт, ошибка уже записана). ^cancel-cause-immutable

С какой версии Go доступен WithCancelCause: Go 1.20+. ^cancel-cause-version

## Связь
- [[Context]] — дерево контекстов, отмена по цепочке
- [[context WithTimeout]] — автоотмена по времени (тоже использует cancel внутри)
- [[WORK-BASE/interviews/theory/Go/Graceful Shutdown]] — signal.NotifyContext для завершения приложения

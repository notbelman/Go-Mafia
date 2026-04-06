- регистрирует функцию, которая выполнится после отмены контекста (Go 1.21+)
- возвращает stop() — можно отменить регистрацию, если ещё не поздно
- удобно для логирования, метрик, cleanup после отмены

---

## Что такое context.AfterFunc

`context.AfterFunc` — регистрирует функцию, которая будет вызвана в отдельной горутине после отмены контекста. Добавлено в Go 1.21. ^af-def

## Паттерн

```go
ctx, cancel := context.WithTimeout(parent, 5*time.Second)
defer cancel()

stop := context.AfterFunc(ctx, func() {
    log.Println("контекст отменён, поднимаю метрику")
    metrics.Inc("context_canceled")
})

// Если нужно отменить регистрацию до отмены контекста:
stopped := stop()  // true = успели, функция НЕ выполнится
                    // false = уже выполняется/выполнилась
```

^af-pattern

## stop() — возвращаемое значение

`AfterFunc` возвращает функцию `stop()`. Вызов `stop()` отменяет регистрацию callback-а. ^af-stop-purpose

`stop()` возвращает `true` — успели отменить, функция НЕ выполнится. `stop()` возвращает `false` — уже поздно: функция выполняется или уже выполнилась. ^af-stop-return

## Версия Go

`context.AfterFunc` появился в Go 1.21. ^af-version

## Когда использовать

Удобно для логирования, метрик и cleanup-операций, которые нужно выполнить после отмены контекста — без необходимости следить за `<-ctx.Done()` вручную. ^af-use-cases

## Связь
- [[context WithCancel]] — cancel() триггерит AfterFunc
- [[context WithTimeout]] — таймаут тоже триггерит AfterFunc

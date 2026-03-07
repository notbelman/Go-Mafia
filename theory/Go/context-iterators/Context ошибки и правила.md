- ctx — всегда первый аргумент, не поле структуры (контекст привязан к вызову, не к объекту)
- defer cancel() всегда, nil вместо контекста = паника, не хранить ctx в struct
- проверять отмену перед значимыми операциями (БД, сеть), не в каждой функции

---

## ctx — первый аргумент, не поле структуры

```go
// ПРАВИЛЬНО: ctx привязан к конкретному вызову
func DoQuery(ctx context.Context, query string) error

// НЕПРАВИЛЬНО: методы структуры вызываются из разных контекстов
type Worker struct {
    ctx context.Context  // какой ctx? от какого запроса?
}
```

Контекст — это не состояние объекта. Это одноразовый неизменяемый объект, привязанный к конкретному запросу. ^ctx-not-state

Хранить его в структуре — антипаттерн (бывают исключения: errgroup и подобные, но это редкость). ^ctx-struct-antipattern

## Где проверять отмену

Не в каждой функции. Перед значимыми операциями — зачем идти в БД, если контекст уже отменён? ^ctx-check-where

```go
func processOrder(ctx context.Context, order Order) error {
    // перед тяжёлой операцией — проверяем
    if ctx.Err() != nil {
        return ctx.Err()
    }
    if err := db.Save(ctx, order); err != nil {
        return err
    }

    // перед следующей тяжёлой операцией
    if ctx.Err() != nil {
        return ctx.Err()
    }
    return notifyService(ctx, order)
}
```

## Обязательные правила

| Правило | Почему |
|:--------|:-------|
| `defer cancel()` всегда | утечка ресурсов (таймеры, горутины) |
| Не передавать `nil` | паника; используй `TODO()` или `Background()` |
| Не хранить в struct | теряется связь с вызовом |
| ctx — первый аргумент | конвенция Go, линтеры |
| cancel вызывать там же где создали | иначе спагетти-код |
| Не разрывать parent-child | потеря Values + отмена не дойдёт |
| WithValue — только метаданные | не бизнес-логику, не зависимости |

^ctx-rules-table

## defer cancel() — утечка ресурсов

Если не вызвать cancel() — утекают ресурсы: таймеры и горутины, которые runtime держит до отмены. Поэтому `defer cancel()` сразу после создания контекста. ^ctx-defer-cancel-why

## nil контекст — паника

Передача nil вместо контекста вызывает панику. Используй `context.TODO()` или `context.Background()` если контекст ещё не определён. ^ctx-nil-panic

## cancel вызывать там же где создали

cancel — это не что-то, что нужно пробрасывать по всей программе. Создал контекст — сразу defer cancel(). Иначе спагетти-код. ^ctx-cancel-locality

## Не разрывать parent-child

Разрыв parent-child приводит к потере Values и к тому, что отмена родителя не дойдёт до детей. ^ctx-parent-child

## WithValue — только метаданные

WithValue предназначен только для метаданных (trace ID, request ID, user ID), не для бизнес-логики и не для передачи зависимостей. ^ctx-withvalue-scope

## Err() — две ошибки

```go
ctx.Err() == context.Canceled         // отменён через cancel()
ctx.Err() == context.DeadlineExceeded // истёк timeout/deadline
```

^ctx-err-two-values

## Связь
- [[Context]] — интерфейс и дерево
- [[context WithoutCancel]] — как не разрывать связь parent-child
- [[context WithValue]] — коллизии ключей и правильные type definitions

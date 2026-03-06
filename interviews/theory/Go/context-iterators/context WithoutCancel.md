- создаёт контекст, который НЕ отменяется при отмене родителя, но сохраняет все Values ^woc-purpose
- зачем: когда часть работы не должна отменяться (логи, метрики, cleanup), но нужны метаданные родителя ^woc-usecase
- НИКОГДА не разрывать parent-child через context.Background() — «не отдавать контекст в детдом» ^woc-never-background

---

## Проблема: нужно продолжить работу после отмены

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()  // отменится когда клиент отключится

    result, err := doWork(ctx)

    // Хотим залогировать даже если клиент ушёл
    // trace ID нужен для лога — он в контексте родителя
    logCtx := context.WithoutCancel(ctx)
    go logRequest(logCtx, result, err)  // не отменится
}
```

Типичный кейс: нужно выполнить cleanup/logging после отмены родительского контекста, но trace ID и другие метаданные должны быть доступны. ^woc-pattern-why

## Почему не context.Background()

```go
// ПЛОХО: разорвали связь parent-child
logCtx := context.Background()
go logRequest(logCtx, result, err)
// logRequest не сможет достать trace ID из контекста!
// сейчас может работать, через год добавят Value — сломается
```

context.Background() разрывает всю цепочку: и отмену, и Values. WithoutCancel обрывает только отмену, Values остаются доступны. Именно для этого его и создали. ^woc-bg-vs-woc

В Яндексе ловили баги: разорвали связь parent-child → часть запросов не отменялась → приходилось чинить. В стандартной библиотеке таких проблем нет, но в сторонних — запросто. ^woc-real-bugs

## Что сохраняется / теряется

```go
parent, cancel := context.WithTimeout(bg, time.Second)
child := context.WithoutCancel(parent)

cancel()

parent.Err()   // context canceled
child.Err()    // nil — не отменён
child.Done()   // nil — никогда не закроется

// Но Values доступны!
child.Value(traceKey{})  // значение из parent
```

| Сохраняется | Теряется |
|:------------|:---------|
| Values (вся цепочка) | Отмена родителя |
| | Deadline |

`child.Done()` возвращает `nil` — канал никогда не закроется. `child.Err()` всегда `nil`. ^woc-done-nil

## Связь
- [[Context]] — дерево контекстов, природа матрёшки
- [[context WithValue]] — Values сохраняются через WithoutCancel
- [[Context ошибки и правила]] — правило: не разрывать parent-child

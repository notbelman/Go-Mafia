- Background — пустой корневой контекст, никогда не отменяется. Единственный допустимый корень дерева ^bg-definition
- TODO — такая же заглушка, но сигнал: «пока не знаю какой контекст использовать, нужен рефакторинг» ^todo-definition
- НИКОГДА не передавать nil вместо контекста — паника (никто не проверяет ctx != nil) ^nil-panic

---

## Когда что

```go
// Background — main, init, тесты, корень дерева
func main() {
    ctx := context.Background()
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    run(ctx)
}

// TODO — временная заглушка, напоминание
func legacyHandler(data string) {
    // TODO: пробросить ctx из вызывающего кода
    doWork(context.TODO(), data)
}
```

^bg-todo-usage

## Почему не nil

```go
doSomething(nil, data)  // паника при обращении к ctx.Err(), ctx.Done()
```

Никто из программистов никогда не пишет `if ctx != nil`. Это не конвенция Go. Если нужна заглушка — Background() или TODO(), но не nil. ^nil-not-convention

## Background vs TODO

Background говорит: «я осознанно использую корневой контекст». TODO говорит: «тут нужно разобраться». ^bg-vs-todo-semantics

На практике мало кто правит TODO в коде, поэтому лучше сразу продумать какой контекст использовать. ^todo-antipattern

## Связь
- [[Context]] — интерфейс и дерево контекстов
- [[Context ошибки и правила]] — почему не nil и другие правила

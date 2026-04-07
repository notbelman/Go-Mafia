- без defer ресурсы **утекают** при панике (файл открыли, паника, Close не вызвался) ^dp-leak-without-defer
- паника долетела до верхушки стека → defer'ы всё равно **вызываются** (перед крашем) ^dp-panic-defers-run
- `runtime.Goexit()` → defer'ы **вызываются**, recover возвращает nil ^dp-goexit-defers
- `os.Exit()` → defer'ы **НЕ вызываются** — процесс убивается сразу ^dp-exit-no-defers
- при graceful shutdown учитывай: os.Exit = никаких defer'ов ^dp-graceful-shutdown

---

## Без defer — утечка ресурсов

```go
func process() {
    f, _ := os.Open("data.txt")
    // ... код ...
    panic("oops")     // файл НЕ будет закрыт!
    f.Close()         // unreachable
}

func caller() {
    defer func() { recover() }()
    process()  // recover сработает, но файл утёк
}
```

Без `defer f.Close()` при панике файл не закрывается, даже если выше по стеку есть recover. ^dp-leak-example

## С defer — безопасно

```go
func process() {
    f, _ := os.Open("data.txt")
    defer f.Close()   // ✅ вызовется даже при панике
    // ... код ...
    panic("oops")     // Close вызовется при раскрутке стека
}
```

Даже если сейчас паники нет — код развивается, nil deref или что-то подобное может появиться. Defer страхует. ^dp-defer-safe

## Паника без recover — defer'ы всё равно работают

```go
func main() {
    defer fmt.Println("deferred!")  // ✅ вызовется перед крашем
    panic("fatal")
}
// deferred!
// panic: fatal
// goroutine 1 [running]: ...
```

В отличие от C++ (без catch деструкторы не вызываются). ^dp-panic-no-recover-cpp

## runtime.Goexit — defer'ы работают

```go
go func() {
    defer fmt.Println("goroutine cleanup")  // ✅ вызовется
    runtime.Goexit()
}()
```

Goexit завершает горутину, defer'ы вызываются. Recover возвращает nil (это не паника). ^dp-goexit-recover-nil

## os.Exit — defer'ы НЕ работают

```go
func main() {
    defer fmt.Println("cleanup")  // ❌ НЕ вызовется
    os.Exit(1)
}
// (ничего не напечатается)
```

os.Exit убивает процесс мгновенно. Никакие defer'ы, flush'и, graceful shutdown — ничего не работает. ^dp-exit-instant

## Сводка

| Ситуация | Defer'ы вызываются? |
|---|---|
| Паника + recover | ✅ |
| Паника без recover (краш) | ✅ |
| runtime.Goexit() | ✅ |
| os.Exit() | ❌ |

^dp-summary-table

## Связь
- [[Паника и recover]] — механика panic/recover
- [[Игнорирование ошибок и ошибки из defer]] — ошибки в defer, подмена
- [[Восстановимые и невосстановимые ошибки]] — OOM = невосстановимо, defer может не помочь

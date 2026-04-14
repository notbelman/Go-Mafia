- defer откладывает выполнение функции до **выхода из функции** (не из блока!) ^dm-scope
- порядок: **LIFO** (стек) — последний defer выполняется первым ^dm-lifo
- под капотом: `runtime.deferproc(fn)` при откладывании, `runtime.deferreturn()` при выходе ^dm-runtime
- недостижимый defer **не планируется** (код должен дойти до строки с defer) ^dm-unreachable
- несколько defer подряд → объединяй в одну анонимную функцию ^dm-combine

---
[[functions Flashcards - defer_mechanics]]
## Привязан к функции, НЕ к блоку

```go
func processFiles(paths []string) {
    for _, p := range paths {
        f, _ := os.Open(p)
        defer f.Close()  // ⚠️ ВСЕ файлы закроются при выходе из функции
                         // а не после каждой итерации!
    }
    // файлы открыты всё время выполнения функции
}
// defer ≠ C++ RAII (деструктор при выходе из scope)
```

**Fix:** вынести тело цикла в отдельную функцию — defer сработает при выходе из неё. ^dm-not-block

## Порядок — LIFO (стек)

```go
defer fmt.Println(3)  // отложен первым
defer fmt.Println(2)  // отложен вторым
defer fmt.Println(1)  // отложен третьим
// LIFO: 1 → 2 → 3
```

Порядок вызова: **обратный** порядку откладывания (как стек). ^dm-lifo-example

## Недостижимый defer

```go
func example() {
    defer fmt.Println("always")  // ✅ выполнится

    if false {
        defer fmt.Println("never")  // ❌ код сюда не дойдёт — defer НЕ запланирован
    }

    return
    defer fmt.Println("also never")  // ❌ после return — недостижимо
}
```

^dm-unreachable-example

## Объединение defer'ов

```go
// ❌ Много отдельных defer
defer mu.Unlock()
defer f.Close()
defer span.End()

// ✅ Один defer — контролируемый порядок
defer func() {
    span.End()
    f.Close()
    mu.Unlock()
}()
```

^dm-combine-example

## Связь
- [[Defer аргументы и ловушки]] — вычисление аргументов, nil-функция
- [[Defer и именованные возвращаемые]] — модификация результата в defer

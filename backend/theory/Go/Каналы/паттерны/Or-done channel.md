- Or-done channel — range по каналу с возможностью прерывания через done-канал
- Без or-done: монструозный select на каждой итерации. С or-done: одна строчка `for val := range orDone(done, ch)`
- Инкапсулирует приоритизацию done внутри себя

---

## Проблема

Хочу итерировать по каналу, но прервать если придёт done: ^ordone-problem

```go
// БЕЗ or-done — монструозная конструкция на каждой итерации:
for {
    select {
    case <-done:
        return
    default:
    }

    select {
    case <-done:
        return
    case val, ok := <-inputCh:
        if !ok { return }
        fmt.Println(val)
    }
}
```

## Решение — or-done

```go
// С or-done — одна строка:
for val := range orDone(done, inputCh) {
    fmt.Println(val)
}
```

^ordone-solution

## Реализация

```go
func orDone(done <-chan struct{}, input <-chan int) <-chan int {
    output := make(chan int)

    go func() {
        defer close(output)
        for {
            // приоритет done
            select {
            case <-done:
                return
            default:
            }

            select {
            case <-done:
                return
            case val, ok := <-input:
                if !ok {
                    return  // input закрыт
                }
                output <- val
            }
        }
    }()

    return output
}
```

^ordone-impl

## Как работает

1. Приходит `done` → горутина выходит → `output` закрывается → `range` завершается
2. Приходит значение из `input` → пишется в `output` → `range` получает
3. `input` закрыт → `ok == false` → горутина выходит → `output` закрывается

Приоритизация done через двойной select. ^ordone-how

## Зачем двойной select

Первый select с `default` — проверка done без блокировки (приоритет отмены). Если done не закрыт — идём во второй select, где ждём либо done либо данные. Это гарантирует что done обработается даже когда input активно поставляет данные. ^ordone-double-select

## Пример

```go
done := make(chan struct{})
ch := make(chan int)

go func() {
    for i := 0; ; i++ {
        ch <- i
        time.Sleep(400 * time.Millisecond)
    }
}()

go func() {
    time.Sleep(1 * time.Second)
    close(done)  // прерываем через секунду
}()

for val := range orDone(done, ch) {
    fmt.Println(val)  // напечатает 0, 1, 2 — потом прервётся
}
```

^ordone-example

## Связь
- [[Done channel]] — паттерн двух каналов (or-done использует первый)
- [[Fan-In]] — or-done = fan-in-подобный паттерн
- [[Pipeline]] — or-done полезен для прерывания стадий pipeline

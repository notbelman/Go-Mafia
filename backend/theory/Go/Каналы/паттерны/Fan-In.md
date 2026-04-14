- Fan-In (merge) — множество каналов сливаются в один: данные из N источников читаются в один результирующий канал
- Паттерн: создать канал → вернуть → в горутинах асинхронно разгребать входные каналы → WaitGroup + close в отдельной горутине
- Типовой сценарий: агрегация данных из нескольких источников

---

## Идея

```
ch1 ──┐
ch2 ──┼──→ merged channel ──→ consumer
ch3 ──┘
```

Все данные из ch1, ch2, ch3 появляются в одном канале. Порядок не гарантирован. ^fanin-idea

## Реализация

```go
func merge(channels ...<-chan int) <-chan int {
    result := make(chan int)
    var wg sync.WaitGroup

    wg.Add(len(channels))
    for _, ch := range channels {
        go func(c <-chan int) {
            defer wg.Done()
            for val := range c {
                result <- val
            }
        }(ch)
    }

    // Закрытие в ОТДЕЛЬНОЙ горутине — не блокируем клиента
    go func() {
        wg.Wait()
        close(result)
    }()

    return result
}
```

## Почему close в отдельной горутине

```go
// НЕПРАВИЛЬНО — deadlock!
wg.Wait()      // блокируемся здесь
close(result)  // никогда не дойдём
return result  // клиент не получит канал → горутины заблокируются на записи
```

Нужно вернуть канал клиенту **сразу**, а закрытие отложить до завершения всех горутин. ^fanin-close-goroutine

## Типовой паттерн (повторяется везде)

```
1. Создать результирующий канал
2. Вернуть его клиенту
3. В отдельных горутинах асинхронно писать в него
4. Закрыть когда все закончили
```

Этот паттерн — основа Fan-In, Fan-Out, Tee, Transform, Filter, Pipeline. ^fanin-pattern

## Использование

```go
ch1, ch2, ch3 := make(chan int), make(chan int), make(chan int)

// продюсеры пишут в разные каналы
go produce(ch1)
go produce(ch2)
go produce(ch3)

// consumer читает из одного
for val := range merge(ch1, ch2, ch3) {
    fmt.Println(val)
}
```

## Гарантии порядка

Порядок прихода значений в merged channel **не гарантирован** — зависит от того, в какой горутине данные появились раньше. ^fanin-order

## Связь
- [[Fan-Out и Tee]] — обратная операция: один канал → много
- [[Pipeline]] — fan-in используется для объединения параллельных стадий
- [[Or-done channel]] — fan-in + возможность прерывания

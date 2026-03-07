- Bridge — канал каналов (`chan chan T`) разворачивается в плоский канал (`chan T`)
- Последовательно вычитывает все данные из первого внутреннего канала, потом из второго и т.д.
- Редкий паттерн, но полезен когда данные приходят пачками через вложенные каналы

---

## Проблема

Есть канал, в который приходят другие каналы. Хочу читать все данные плоско: ^bridge-problem

```go
// Имеем:
chanOfChans := make(chan (<-chan string))

// Хотим:
for val := range bridge(chanOfChans) {
    fmt.Println(val)  // все строки последовательно
}
```

## Реализация

```go
func bridge(chanOfChans <-chan (<-chan string)) <-chan string {
    output := make(chan string)

    go func() {
        defer close(output)
        for ch := range chanOfChans {      // для каждого внутреннего канала
            for val := range ch {           // вычитать все значения
                output <- val
            }
        }
    }()

    return output
}
```

^bridge-impl

## Пример использования

```go
chanOfChans := make(chan (<-chan string))

go func() {
    defer close(chanOfChans)

    ch1 := make(chan string, 2)
    ch1 <- "a"
    ch1 <- "b"
    close(ch1)

    ch2 := make(chan string, 2)
    ch2 <- "c"
    ch2 <- "d"
    close(ch2)

    chanOfChans <- ch1
    chanOfChans <- ch2
}()

for val := range bridge(chanOfChans) {
    fmt.Println(val)  // a, b, c, d
}
```

^bridge-example

## Особенности

Вычитывание **последовательное**: сначала весь ch1, потом весь ch2. ^bridge-sequential

Можно улучшить: параллельное вычитывание (горутина на каждый внутренний канал + fan-in). ^bridge-parallel-opt

Внутренние каналы должны быть закрыты, иначе `range ch` зависнет навечно. ^bridge-close-required

## Когда нужен

- Батчи данных приходят через канал каналов
- Пагинация: каждая страница = канал результатов
- Стриминг: каждый chunk = отдельный канал ^bridge-when

## Связь
- [[Fan-In]] — merge каналов (параллельно). Bridge — последовательно
- [[Pipeline]] — bridge может быть стадией pipeline

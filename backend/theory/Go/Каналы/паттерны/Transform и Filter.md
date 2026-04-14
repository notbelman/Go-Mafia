- Transform — декоратор канала: читает значения, преобразует, пишет в новый канал
- Filter — декоратор канала: пропускает только значения, прошедшие проверку
- Оба — строительные блоки для Pipeline: компонуются цепочкой

---

## Transform

Канал-посредник, который преобразует каждое значение: ^transform-def

```go
func transform(input <-chan int, fn func(int) int) <-chan int {
    output := make(chan int)
    go func() {
        defer close(output)
        for val := range input {
            output <- fn(val)
        }
    }()
    return output
}
```

Использование:

```go
ch := gen(1, 2, 3, 4, 5)
squared := transform(ch, func(x int) int { return x * x })

for val := range squared {
    fmt.Println(val)  // 1, 4, 9, 16, 25
}
```

## Filter

Канал-посредник, который пропускает только подходящие значения: ^filter-def

```go
func filter(input <-chan int, predicate func(int) bool) <-chan int {
    output := make(chan int)
    go func() {
        defer close(output)
        for val := range input {
            if predicate(val) {
                output <- val
            }
            // не подходит — отбрасываем
        }
    }()
    return output
}
```

Использование:

```go
ch := gen(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
odd := filter(ch, func(x int) bool { return x%2 != 0 })

for val := range odd {
    fmt.Println(val)  // 1, 3, 5, 7, 9
}
```

## Композиция — декларативный стиль

Transform и Filter — декораторы. Принимают канал, возвращают канал. Можно вкладывать: ^transform-filter-compose

```go
result := filter(
    transform(
        gen(1, 2, 3, 4, 5),
        func(x int) int { return x * x },     // возвести в квадрат
    ),
    func(x int) bool { return x > 10 },       // оставить > 10
)
// результат: 16, 25
```

Читается как декларативное описание: "сгенерируй → возведи в квадрат → отфильтруй больше 10".

## Общий паттерн

Обе функции следуют одному шаблону: ^transform-filter-pattern

```
1. Принять входной канал
2. Создать выходной канал
3. Вернуть выходной канал
4. В горутине: range по входному → логика → запись в выходной → close
```

Это тот же паттерн что в Fan-In, Fan-Out, Tee — **создал, вернул, асинхронно процессишь**.

## Применение Filter

Filter применяется для: ^filter-usecases

- Отбросить секретные/персональные данные перед логированием
- Rate limiting (пропускать только N в секунду)
- Валидация (отбрасывать невалидные сообщения)
- Дедупликация

## Связь
- [[Pipeline]] — цепочка transform/filter = pipeline
- [[Fan-In]] — тот же паттерн создания канала
- [[Fan-Out и Tee]] — тот же паттерн создания канала

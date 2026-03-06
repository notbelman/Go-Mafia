- Generator — ленивая генерация значений: следующее значение вычисляется только когда его запросят
- Два способа в Go: замыкание (closure) или канал
- Канальный генератор = горутина пишет в канал, потребитель читает range'ом

---

## Способ 1: замыкание

```go
func counter(start int) func() int {
    n := start
    return func() int {
        result := n
        n++
        return result
    }
}

gen := counter(10)
fmt.Println(gen())  // 10
fmt.Println(gen())  // 11
fmt.Println(gen())  // 12
```

Переменная `n` замкнута — живёт между вызовами. Каждый вызов инкрементирует. ^gen-closure

## Способ 2: канал

```go
func rangeGen(start, end int) <-chan int {
    ch := make(chan int)
    go func() {
        defer close(ch)
        for i := start; i < end; i++ {
            ch <- i  // блокируется пока не прочитают
        }
    }()
    return ch
}

for val := range rangeGen(100, 200) {
    fmt.Println(val)
}
```

**Ленивость**: горутина блокируется на `ch <- i` пока потребитель не прочитает. Следующее значение генерируется только по запросу. ^gen-channel-lazy

## Замыкание vs Канал

```
              Замыкание            Канал
Вызов         gen()                range ch / <-ch
Конкурентно   нет (не safe)        да (канал safe)
Завершение    нет сигнала          close(ch) → range выходит
Горутина      нет                  да (overhead)
Бесконечный   да (пока вызывают)   да (пока читают)
``` ^gen-compare

Замыкание проще если один потребитель. Канал — если нужна конкурентность или интеграция с select/pipeline. ^gen-when

## Бесконечный генератор

```go
func fibonacci() <-chan int {
    ch := make(chan int)
    go func() {
        a, b := 0, 1
        for {
            ch <- a
            a, b = b, a+b
        }
        // не закрываем — бесконечный
    }()
    return ch
}

fib := fibonacci()
for i := 0; i < 10; i++ {
    fmt.Println(<-fib)
}
// горутина утечёт! нужен done channel для остановки
```

Для бесконечных генераторов нужен механизм остановки (done channel или context). ^gen-infinite-leak

## Связь
- [[Transform и Filter]] — генератор часто первая стадия pipeline (gen → transform → ...)
- [[Pipeline]] — gen() = первая стадия
- [[Promise и Future]] — тоже абстракция поверх каналов
- [[Done channel]] — остановка бесконечного генератора

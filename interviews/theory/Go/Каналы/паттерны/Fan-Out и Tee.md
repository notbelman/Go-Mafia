- Fan-Out (split) — один канал → много, каждое значение идёт в **один** из выходных каналов (round-robin)
- Tee — один канал → много, каждое значение **дублируется** во все выходные каналы
- Fan-Out = ИЛИ (распределение нагрузки), Tee = И (репликация данных)

---

## Fan-Out

```
               ┌──→ ch_out[0]  (получает 0, 2, 4)
input ─────────┤
               └──→ ch_out[1]  (получает 1, 3, 5)
```

Значение попадает **либо** в один, **либо** в другой канал. Не дублируется. ^fanout-def

```go
func split(input <-chan int, n int) []<-chan int {
    outputs := make([]chan int, n)
    for i := range outputs {
        outputs[i] = make(chan int)
    }

    go func() {
        defer func() {
            for _, ch := range outputs { close(ch) }
        }()
        i := 0
        for val := range input {
            outputs[i%n] <- val  // round-robin
            i++
        }
    }()

    // convert []chan to []<-chan
    result := make([]<-chan int, n)
    for i, ch := range outputs { result[i] = ch }
    return result
}
```

**Нюанс**: если один из выходных каналов забит (потребитель тормозит), запись блокируется. Решение — неблокирующая запись через select + default, чтобы пойти в другой канал. ^fanout-blocking

**Когда использовать**: распределение по шардам, балансировка между воркерами. ^fanout-when

## Tee

```
               ┌──→ ch_out[0]  (получает 0, 1, 2, 3, 4)
input ─────────┤
               └──→ ch_out[1]  (получает 0, 1, 2, 3, 4)
```

Каждое значение дублируется во **все** выходные каналы. ^tee-def

```go
func tee(input <-chan int) (<-chan int, <-chan int) {
    out1 := make(chan int)
    out2 := make(chan int)

    go func() {
        defer close(out1)
        defer close(out2)
        for val := range input {
            out1 <- val
            out2 <- val  // то же значение — в оба
        }
    }()

    return out1, out2
}
```

**Когда использовать**: репликация во все шарды, запись в лог + обработка, аудит + основная логика. ^tee-when

## Поведение Tee при блокировке

В Tee горутина пишет сначала в out1, потом в out2 последовательно. Если любой из потребителей тормозит — **блокируется доставка всем**. ^tee-blocking

## Сравнение

```
              Fan-Out              Tee
Значение      в ОДИН из каналов    во ВСЕ каналы
Оператор      ИЛИ                  И
Сценарий      балансировка          репликация
Блокировка    один канал забит →    любой канал забит →
              блокировка            блокировка всех
``` ^fanout-tee-compare

## Связь
- [[Fan-In]] — обратная операция: много → один
- [[Pipeline]] — fan-out для параллелизации тормозящей стадии

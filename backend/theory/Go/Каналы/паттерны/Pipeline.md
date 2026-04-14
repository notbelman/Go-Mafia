- Pipeline — цепочка стадий обработки: gen → stage1 → stage2 → ... → consumer. Каждая стадия = горутина + канал
- Декларативный код: `for val := range multiply(multiply(gen(1,2,3))) { ... }`
- Тормозящую стадию можно параллелить: добавить горутин + fan-out/fan-in

---

## Идея

Данные текут через цепочку стадий. Каждая стадия читает из входного канала, обрабатывает, пишет в выходной: ^pipeline-idea

```
gen(1,2,3) ──→ parse() ──→ validate() ──→ send() ──→ consumer
     канал        канал         канал          канал
```

## Реализация

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func multiply(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for val := range in {
            out <- val * val
        }
    }()
    return out
}
```

Использование — компактно и декларативно: ^pipeline-declarative

```go
for val := range multiply(multiply(gen(1, 2, 3, 4))) {
    fmt.Println(val)
}
```

## Параллелизация тормозящей стадии

Если одна стадия медленнее остальных — добавляем горутин: ^pipeline-parallelize

```
               ┌─ send() горутина 1 ─┐
parse() ───────┤                      ├──→ merged output
               └─ send() горутина 2 ─┘
```

```go
func parallelSend(in <-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup

    numWorkers := 2  // конфигурируем
    wg.Add(numWorkers)
    for i := 0; i < numWorkers; i++ {
        go func() {
            defer wg.Done()
            for val := range in {
                time.Sleep(100 * time.Millisecond) // тяжёлая отправка
                out <- val
            }
        }()
    }

    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

Несколько горутин читают из одного канала (safe — канал потокобезопасен). Fan-in на выходе через общий канал + WaitGroup. ^pipeline-fanin-wg

## Реальный пример (из лекции)

Сервис профилирования: ^pipeline-real-example
```
входной канал профилей → parse (1 горутина) → канал → send to DB (N горутин)
```
Стадия send тормозила → добавили горутин → пропускная способность выросла.

## Компромиссы

Плюсы: декларативность, изоляция стадий, лёгкая параллелизация узких мест. ^pipeline-pros

Минусы: дополнительная синхронизация на каждой стадии, сложнее дебаг, overhead на каналы. ^pipeline-cons

Не нужно строить pipeline из всего. Используй когда обработка **реально** стадийная и стадии можно независимо масштабировать. ^pipeline-when

## Связь
- [[Transform и Filter]] — строительные блоки стадий
- [[Fan-In]] — объединение параллельных воркеров стадии
- [[Fan-Out и Tee]] — распределение между воркерами стадии
- [[Barrier]] — тоже стадии, но с синхронизацией всех горутин

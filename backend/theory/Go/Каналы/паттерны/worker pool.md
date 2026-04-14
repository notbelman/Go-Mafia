- Worker Pool — N горутин-воркеров читают задачи из общего канала. Канал = очередь, N воркеров = ограничение параллелизма
- Паттерн: создать канал задач → запустить N воркеров (`range jobs`) → отправить задачи → `close(jobs)` → WaitGroup.Wait
- Альтернатива "горутина на запрос": worker pool даёт **backpressure** и **контроль ресурсов** (CPU, коннекты к БД)

---

## Идея

```
producer ──→ [jobs channel] ──→ worker 1 ──→ [results channel] ──→ consumer
                             ──→ worker 2 ──→
                             ──→ worker 3 ──→
```

N воркеров конкурентно читают из одного канала (safe — канал потокобезопасен). Каждое значение достаётся **одному** воркеру. ^wp-idea

## Реализация

```go
func worker(id int, jobs <-chan Job, results chan<- Result) {
    for job := range jobs {       // выход когда jobs закрыт
        results <- process(job)
    }
}

func main() {
    jobs := make(chan Job, 100)       // буферизированный — backpressure
    results := make(chan Result, 100)

    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            worker(id, jobs, results)
        }(i)
    }

    // Отправка задач
    for _, j := range taskList {
        jobs <- j  // блокируемся если буфер полон — backpressure
    }
    close(jobs)  // сигнал воркерам: задач больше не будет

    // Закрыть results когда все воркеры завершились
    go func() {
        wg.Wait()
        close(results)
    }()

    for res := range results {
        fmt.Println(res)
    }
}
```

^wp-impl

## Почему close(results) в отдельной горутине

Тот же паттерн что в Fan-In: нужно вернуть/использовать канал **сразу**, а закрытие отложить до завершения всех горутин. Иначе deadlock: `wg.Wait()` блокирует, `range results` не начнётся, воркеры заблокируются на записи. ^wp-close-goroutine

## Сколько воркеров

|Тип нагрузки|Рекомендация|
|:--|:--|
|**CPU-bound**|`runtime.NumCPU()` — больше бессмысленно|
|**IO-bound**|десятки-сотни — горутины спят на I/O|
|**Ограниченный ресурс**|по числу ресурсов (коннекты к БД)|

^wp-sizing

## Worker Pool vs горутина на запрос

```go
// Горутина на запрос — просто, но без контроля
for _, req := range requests {
    go handle(req)  // 1M запросов = 1M горутин
}

// Worker pool — контролируемый параллелизм
for _, req := range requests {
    jobs <- req  // буфер полон? блокируемся — backpressure
}
```

**Backpressure**: буферизированный канал + N воркеров = продюсер замедляется если воркеры не успевают. Горутина-на-запрос не имеет backpressure — память растёт бесконтрольно. ^wp-backpressure

## Graceful shutdown

```go
// Воркер с контекстом
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            return
        case job, ok := <-jobs:
            if !ok { return }
            process(ctx, job)
        }
    }
}
```

^wp-shutdown

## Связь

- [[Fan-In]] — тот же паттерн: N горутин → один канал → WaitGroup + close в отдельной горутине
- [[Fan-Out и Tee]] — fan-out раздаёт задачи, worker pool читает из общего канала
- [[Pipeline]] — worker pool = параллелизация тормозящей стадии pipeline
- [[Семафор]] — семафор через канал = worker pool без отдельных горутин
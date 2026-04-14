**Concurrency (конкурентность)** — структура программы. Много задач управляются одновременно, но могут выполняться по очереди на одном ядре.

**Parallelism (параллельность)** — выполнение. Задачи реально работают одновременно на разных ядрах.
```
GOMAXPROCS=1 (concurrency, нет parallelism):

  Core 0:  [G1][G2][G1][G3][G2][G1]...
           горутины чередуются на одном ядре

GOMAXPROCS=4 (concurrency + parallelism):

  Core 0:  [G1][G1][G1]...
  Core 1:  [G2][G2][G2]...
  Core 2:  [G3][G3][G3]...
  Core 3:  [G4][G4][G4]...
           горутины реально работают одновременно
```

> "Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once." — Rob Pike

Go по умолчанию даёт оба: concurrency через горутины, parallelism через GOMAXPROCS = NumCPU.
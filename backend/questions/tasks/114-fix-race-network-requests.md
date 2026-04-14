---
type: task
companies:
  - Napoleon IT
topic: Go
subtopic:
  - Concurrency
  - Race Condition
  - WaitGroup
  - Worker Pool
title: Исправить race condition и распараллелить запросы
---

## Условие

Найти баги в коде, исправить, распараллелить. Сделать worker pool.

## Пример
```go
var count int
const numRequests = 10000

func main() {
	for i := 0; i < numRequests; i++ {
		networkRequest()
	}
	fmt.Println(count)
}

func networkRequest() {
	time.Sleep(time.Millisecond) //эмуляция сетевого запроса
	count++
}
```

## Решение

Два бага:
1. **Race condition** — `count++` не атомарная операция (read → increment → write), конкурентные горутины затирают друг друга.
2. **Последовательное выполнение** — 10000 запросов по 1мс = 10 секунд вместо параллельного выполнения.

### Исправление: горутины + atomic

```go
var count atomic.Int64
const numRequests = 10000

func main() {
	var wg sync.WaitGroup

	for i := 0; i < numRequests; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			networkRequest()
		}()
	}

	wg.Wait()
	fmt.Println(count.Load())
}

func networkRequest() {
	time.Sleep(time.Millisecond)
	count.Add(1)
}
```

### Доп. вопрос: worker pool

10000 горутин — 10000 × ~4KB стека = ~40MB. Worker pool ограничивает конкурентность:

```go
func main() {
	const numRequests = 10000
	const numWorkers = 100

	var wg sync.WaitGroup
	jobs := make(chan struct{}, numRequests)

	// фиксированное количество воркеров
	for i := 0; i < numWorkers; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for range jobs {
				networkRequest()
			}
		}()
	}

	// отправляем задачи
	for i := 0; i < numRequests; i++ {
		jobs <- struct{}{}
	}
	close(jobs)

	wg.Wait()
	fmt.Println(count.Load())
}
```

Worker pool vs горутина-на-запрос: контролируемое потребление памяти, не перегружаем downstream при внешних запросах (БД, HTTP).

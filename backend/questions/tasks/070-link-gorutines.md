---
type: task
companies:
  - LAMODA
topic: Go
subtopic:
  - Goroutines
  - Channels
  - Semaphore
  - HTTP
title: Параллельный обход URL с ограничением одновременных запросов
---

## Условие

Требуется обойти все URL и получить слайс с кодами ответов в том же порядке. Хотим делать параллельно, но не больше k запросов одновременно.

## Пример
```go
var urls = []string{
    "https://www.lamoda.com/p/mp002xw01vkd/clothes-tomollyfromjames-plate/",
    "https://www.lamoda.com/p/mp002xw14uf2/clothes-tomollyfromjames-plate/",
    "https://www.lamoda.com/p/rtladr746901/clothes-iceberg-plate/",
    // ...
}

func crawl(urls []string, k int) []int {
}

func main() {
    result := crawl(urls, 5)
    fmt.Println("All done")
}
```

## Решение
```go
```
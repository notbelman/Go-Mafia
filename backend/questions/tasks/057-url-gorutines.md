---
type: task
companies:
  - МВидео
topic: Go
subtopic:
  - Goroutines
  - Error Handling
  - Context
title: Анализ кода с горутинами для обработки URL — проблемы и улучшения
---

## Условие

Что не так в коде программы, которая запускает горутины для обработки списка URL? Что произойдёт если в слайсе миллион URL? Как завершить все горутины при первой ошибке?

## Пример
```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    urls := []string{
        "https://www.yandex.ru",
        "https://www.mail.ru",
        "https://www.google.com",
    }

    for _, url := range urls {
        go func(url string) {
            fmt.Printf("Fetching %s...\n", url)
            err := fetchUrl(url)
            if err != nil {
                fmt.Printf("Error fetching %s: %v\n", url, err)
                return
            }
            fmt.Printf("Fetched %s\n", url)
        }(url)
    }

    fmt.Println("All requests launched!")
    time.Sleep(400 * time.Millisecond)
    fmt.Println("Program finished.")
}

func fetchUrl(url string) error {
    err := http.Get(url)
    return err
}
```

## Решение
```go
```
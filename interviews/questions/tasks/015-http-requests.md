---
type: task
companies:
  - OZON
topic: Go
subtopic:
  - HTTP
  - net/http
title: Последовательные HTTP запросы по списку URL
explained: "true"
---

## Условие
Напишите программу, которая последовательно выполняет HTTP запросы по предложенному списку ссылок. 
- В случае получения HTTP-кода ответа "200 OK" напечатаем на экране "адрес url - ok". 
- В случае получения кода отличного от "200 OK" или ошибки, выводим "адрес url - not ok".

## Решение

```go
import (
    "fmt"
    "net/http"
)

func main() {
    var urls = []string{
        "http://ozon.ru",
        "https://ozon.ru",
        "http://google.com",
        "http://somesite.com",
        "http://www.not-existend.domain.tld",
        "https://ya.ru",
        "http://ya.ru",
        "http://eeee",
    }
    
    for _, url := range urls {
        resp, err := http.Get(url)
        if err != nil || resp.StatusCode != 200 {
            fmt.Printf("%s - not ok\n", url)
        } else {
            fmt.Printf("%s - ok\n", url)
        }
    }
}
```

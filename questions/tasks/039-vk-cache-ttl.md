---
type: task
companies:
  - VK
topic: Go
subtopic:
  - Cache
  - TTL
  - Map
  - Concurrency
title: Реализовать кэш с TTL для User
---

## Условие
Написать реализацию кэша с TTL (time to live) для структуры User. В качестве ключа использовать `User.ID`.

```go
type User struct {
    ID   string
    Name string
}
```

## Решение

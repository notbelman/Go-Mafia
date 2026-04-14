---
type: task
companies:
  - X5
  - OZON
topic: Go
subtopic:
  - Map
  - Linked List
  - Concurrency
title: Реализация LRU-кэша с доступом за O(1)
---

## Условие

Реализовать кэш с доступом к ключу за O(1). При превышении ёмкости вытесняется наименее недавно использованный элемент (LRU — Least Recently Used).

Интерфейс:
- `Get(key string) string` — получить значение по ключу
- `Put(key string, value string)` — положить значение по ключу

## Пример
```go
package main

type LRUCache struct {}

func (c *LRUCache) Get(key string) string {}

func (c *LRUCache) Put(key string, value string) {}
```

## Решение
```go
```
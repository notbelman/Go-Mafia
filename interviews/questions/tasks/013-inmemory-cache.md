---
type: task
companies:
  - OZON
topic: Go
subtopic:
  - Map
  - Mutex
  - Interfaces
title: Реализовать in-memory cache
explained: "true"
---

## Условие
Нужно написать простую библиотеку in-memory cache.
Для простоты считаем, что у нас бесконечная память и нам не нужно задумываться об удалении ключей.

Реализация должна удовлетворять интерфейсу:

```go
type Cache interface {
    Set(k, v string)
    Get(k string) (v string, ok bool)
}
```

## Решение

```go
type Cache interface {
    Set(k, v string)
    Get(k string) (v string, ok bool)
}

type InMemoryCache struct {
    data map[string]string
    mu   sync.RWMutex
}

func NewInMemoryCache() *InMemoryCache {
    return &InMemoryCache{data: make(map[string]string)}
}

func (cache *InMemoryCache) Set(k, v string) {
    cache.mu.Lock()
    defer cache.mu.Unlock()
    cache.data[k] = v
}

func (cache *InMemoryCache) Get(k string) (string, bool) {
    cache.mu.RLock()
    defer cache.mu.RUnlock()
    v, ok := cache.data[k]
    return v, ok
}
```

**Ключевые моменты:**
- `sync.RWMutex` — позволяет множественные чтения, но эксклюзивную запись
- `RLock/RUnlock` для Get — несколько горутин могут читать одновременно
- `Lock/Unlock` для Set — только одна горутина может писать

### Что выбрать sync.Map или map + RWMutex/Mutex
[[sync.Map vs map + RWMutex — Когда Что Использовать]]

### Что выбрать map + RWMutex или просто Mutex
[[sync.Mutex vs sync.RWMutex]]
---
type: task
companies:
  - MTS
topic: Go
subtopic:
  - Mutex
  - RWMutex
  - Singleflight
  - Hot Key
  - Channels
  - Cache
title: ProxyStore — оптимизация метода Get с проблемой горячего ключа
---

## Условие

Дан ProxyStore с in-memory кэшем и внешним KVStore. Метод Get имеет слабые стороны. Нужно найти проблемы и переписать, решив проблему горячего ключа (когда множество горутин одновременно запрашивают один и тот же ключ).

## Пример
```go
type KVStore interface {
    // Get value by key.
    // Used very often.
    Get(key string) (string, error)

    // Get available keys.
    // Used rarely.
    Keys() ([]string, error)
}

type ProxyStore struct {
    imMemoryCache map[string]string
    cacheMx       sync.RWMutex
    kv            KVStore
}

func (p *ProxyStore) Get(key string) (string, error) {
    p.cacheMx.Lock()
    val, ok := p.imMemoryCache[key]
    p.cacheMx.Unlock()
    if ok {
        return val, nil
    }

    p.cacheMx.Lock()
    val, err := p.kv.Get(key)
    if err != nil {
        return "", err
    }
    p.imMemoryCache[key] = val
    p.cacheMx.Unlock()

    return val, nil
}
```

Вопросы:
1. Какие слабые стороны у текущей реализации?
2. Как решить проблему горячего ключа (singleflight)?
3. Как реализовать ожидание других горутин через каналы?
4. Как разослать уведомление всем ожидающим?

## Решение
```go
```
---
type: task
companies:
  - Napoleon IT
topic: Go
subtopic:
  - Concurrency
  - sync.RWMutex
  - Map
title: Потокобезопасная мапа с GetOrCreate
---

## Условие

Реализовать потокобезопасный метод `GetOrCreate` — если ключ есть, вернуть значение; если нет — сохранить переданное и вернуть его.

## Пример
```go
type SafeMap struct {
	data map[string]string
}

func (s *SafeMap) GetOrCreate(key, value string) string {
}
```

## Решение

Double-checked locking через `sync.RWMutex` — сначала дешёвый `RLock`, если ключа нет — апгрейд до `Lock` с повторной проверкой.

```go
type SafeMap struct {
	mu   sync.RWMutex
	data map[string]string
}

func (s *SafeMap) GetOrCreate(key, value string) string {
	// сначала пробуем читать — RLock дешевле
	s.mu.RLock()
	if v, ok := s.data[key]; ok {
		s.mu.RUnlock()
		return v
	}
	s.mu.RUnlock()

	// ключа нет — берём эксклюзивную блокировку
	s.mu.Lock()
	defer s.mu.Unlock()

	// double-check: пока ждали Lock другая горутина могла создать ключ
	if v, ok := s.data[key]; ok {
		return v
	}

	s.data[key] = value
	return value
}
```

### Почему double-check обязателен

Между `RUnlock` и `Lock` — окно, в котором другая горутина могла уже вставить тот же ключ. Без повторной проверки после `Lock` — перезапишем значение.

### Трассировка

G1 и G2 одновременно вызывают `GetOrCreate("x", "hello")`, ключа нет:
1. Оба прошли `RLock` → ключа нет → `RUnlock`
2. G1 взял `Lock` первым → double-check → ключа нет → вставил `"x"="hello"` → вернул `"hello"`
3. G2 взял `Lock` → double-check → ключ уже есть → вернул `"hello"` без перезаписи

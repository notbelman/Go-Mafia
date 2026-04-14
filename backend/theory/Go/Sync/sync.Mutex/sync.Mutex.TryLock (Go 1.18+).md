## Что делает
```go
func (m *Mutex) TryLock() bool
```

Пытается захватить лок **без блокировки**: ^trylock-what
- `true` — захватили, нужен Unlock()
- `false` — занято, сразу вернулись

## Когда возвращает false

- Лок уже захвачен ^trylock-false-locked
- Mutex в starvation mode (даже если свободен!) ^trylock-false-starving

## Реализация
```go
// упрощённо
func (m *Mutex) TryLock() bool {
    return atomic.CAS(&m.state, 0, mutexLocked)
}
```

Один CAS, никакого спиннинга, никакого ожидания. ^trylock-impl

## Memory model

- Успешный TryLock = Lock (synchronizes-before) ^trylock-memmodel-success
- Неудачный TryLock — **никаких гарантий** ^trylock-memmodel-fail

## ⚠️ Официальное предупреждение

> "Note that while correct uses of TryLock do exist, they are rare, and use of TryLock is often a sign of a deeper problem."

^trylock-warning

## Редкие легитимные кейсы

- Connection pool: "занят ли этот коннект?" ^trylock-usecase-pool
- Избежание deadlock при захвате нескольких локов ^trylock-usecase-deadlock
- Optimistic попытка перед fallback ^trylock-usecase-optimistic

## Добавлено в Go 1.18 ^trylock-version

## Связь
- [[sync.Mutex/Lock()]] — обычный захват vs TryLock
- [[Два режима]] — TryLock возвращает false если Starving
- [[Livelock и Starvation]] — TryLock в цикле = livelock

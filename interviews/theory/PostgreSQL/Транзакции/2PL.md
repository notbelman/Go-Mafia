- 2PL (Two-Phase Locking) — блокирующий concurrency control. Фаза expanding: захватываем блокировки, не отпускаем. Фаза shrinking: после commit/rollback отпускаем все
- Мелкогранулярные RW-блокировки на каждый ключ. Читающие не мешают читающим (shared), пишущие блокируют всех (exclusive)
- Проблема: deadlock. Решения: не бороться, таймаут, wait-for graph

---

## Две фазы

```
Фаза Expanding (захват):
  Lock(key1) → get/set → Lock(key2) → get/set → Lock(key3) → set
  ← блокировки ТОЛЬКО захватываются, НИКОГДА не отпускаются

Commit / Rollback

Фаза Shrinking (освобождение):
  Unlock(key3) → Unlock(key2) → Unlock(key1)
  ← блокировки ТОЛЬКО отпускаются
```

**Почему нельзя отпускать раньше**: если отпустил key1 и потом снова читаешь key1 — между этим другая транзакция могла его изменить → нарушение изоляции.

## RW-блокировки

**GET** → shared lock (RLock). Несколько читающих транзакций не мешают друг другу.

**SET** → exclusive lock (Lock). Блокирует и читающих, и пишущих.

**Повышение ранга**: транзакция сначала GET (shared), потом SET (exclusive) на тот же ключ → нужно RUnlock + Lock. Между ними может встроиться другая транзакция. Для безопасности нужен атомарный upgrade (кастомный RWMutex).

## Архитектура (учебная)

```go
type Scheduler struct {
    locks   map[string]*Lock    // ключ → блокировка
    storage Storage
}

type Lock struct {
    lockType      int           // shared / exclusive
    transactionID int
    mu            sync.RWMutex
}
```

Транзакция на каждый set/get идёт в scheduler → scheduler берёт блокировку на ключ → выполняет операцию в storage. При commit/rollback — обходит operations в обратном порядке, разблокирует.

Rollback: откатывает значения в storage на предыдущие (сохранённые в operations).

## Deadlock

```
Транзакция 1: Lock(key1) ✓ → Lock(key2) ← BLOCKED (key2 занят T2)
Транзакция 2: Lock(key2) ✓ → Lock(key1) ← BLOCKED (key1 занят T1)
→ DEADLOCK
```

**Решения:**

| Подход                                        | Плюс                 | Минус                                 |
| --------------------------------------------- | -------------------- | ------------------------------------- |
| Не бороться (ответственность на программисте) | 0 overhead           | deadlock'и возможны                   |
| Таймаут (убить одну транзакцию)               | Минимальный overhead | Долгое ожидание до таймаута           |
| Wait-for graph (граф ожидания)                | Быстрая реакция      | Overhead на построение и анализ графа |

## Связь
- [[Транзакция и ACID]] — 2PL решает все аномалии на serializable
- [[MVCC]] — альтернативный подход (оптимистичный)
- [[Алгоритмы синхронизации списков]] — тонкая синхронизация = мини-2PL на узлах списка
- [[WORK-BASE/interviews/theory/Go/Concurrency Glossary/Deadlock]] — граф ожидания, порядок захвата

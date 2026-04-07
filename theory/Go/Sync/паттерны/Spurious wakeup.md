- Spurious wakeup — поток просыпается из ожидания **без** сигнала. Просто так. Это разрешено спецификацией
- Поэтому условие ожидания ВСЕГДА проверяют в **цикле while**, не в if
- В Go: каналы и sync.Cond подвержены, но каналы защищают автоматически

---

## Что происходит

Поток ждёт на condition variable (или аналоге). Никто не вызывал Signal/Broadcast. Но поток просыпается.

```
Поток: "Жду пока data != nil"
          ...
          *просыпается*
          data == nil ???  // никто не сигналил!
```

^spurious-def

## Почему это существует

1. **OS уровень**: POSIX разрешает spurious wakeup для `pthread_cond_wait`. Реализация на Linux (futex) может разбудить поток при перебалансировке, обработке сигналов, или из-за внутренней оптимизации ядра ^spurious-reason-os
2. **Производительность**: запрет spurious wakeup потребовал бы дополнительной синхронизации в ядре — дороже для всех ради редкого случая ^spurious-reason-perf
3. **Simplicity**: проще разрешить и сказать "проверяйте в цикле" ^spurious-reason-simplicity

## Правильный паттерн — while, не if

```go
// НЕПРАВИЛЬНО — if
mu.Lock()
if !condition {
    cond.Wait()  // spurious wakeup → продолжаем с невыполненным условием!
}
doWork()
mu.Unlock()

// ПРАВИЛЬНО — for
mu.Lock()
for !condition {
    cond.Wait()  // spurious wakeup → проверяем условие снова
}
doWork()
mu.Unlock()
```

^spurious-for-pattern

Это классический паттерн для `sync.Cond`:

```go
var mu sync.Mutex
var cond = sync.NewCond(&mu)
var queue []int

func consumer() {
    mu.Lock()
    defer mu.Unlock()
    
    for len(queue) == 0 {  // FOR, не IF
        cond.Wait()
    }
    item := queue[0]
    queue = queue[1:]
    // обработать item
}

func producer(item int) {
    mu.Lock()
    queue = append(queue, item)
    mu.Unlock()
    cond.Signal()
}
```

^spurious-sync-cond-example

## Ещё причина для цикла — stolen wakeup

Даже без spurious wakeup цикл нужен:

```
Поток A: ждёт (очередь пуста)
Поток B: ждёт (очередь пуста)
Producer: кладёт 1 элемент, Signal → будит A
Поток C: (не ждал) захватывает мьютекс первым, забирает элемент
Поток A: просыпается, мьютекс свободен, заходит... а очередь пуста!
```

Это **stolen wakeup** — другой поток украл работу между Signal и пробуждением. Цикл `for` спасает и от этого. ^stolen-wakeup

## В Go

**Каналы**: `<-ch` не подвержен spurious wakeup — runtime гарантирует что пробуждение = данные в канале. ^go-channels-safe

**sync.Cond**: подвержен. Документация Go явно говорит: `Wait` может вернуться без Broadcast/Signal. Всегда `for`, никогда `if`. ^go-synccond-spurious

**select**: не подвержен — runtime проверяет условия атомарно. ^go-select-safe

## Связь
- [[Семафор]] — Acquire может spurious wakeup (зависит от реализации)
- [[Примитивный мьютекс]] — park/unpark и проблемы пробуждения
- [[Мьютекс Петерсона]] — memory model проблемы

- Error Group — группа горутин: если одна вернула ошибку, последующие не запускаются
- Реализация: WaitGroup + done-канал (закрывается при первой ошибке) + sync.Once
- Полная реализация с контекстами — `golang.org/x/sync/errgroup`

---

## Идея

Запускаю N горутин. Если одна упала с ошибкой — остальные запускать бессмысленно (например, запросы в шарды: неполные данные не нужны). ^eg-idea

```go
g := NewGroup()

g.Go(func() error {
    return doQuery(shard1)  // если ошибка → остальные не запустятся
})
g.Go(func() error {
    return doQuery(shard2)
})
g.Go(func() error {
    return doQuery(shard3)
})

if err := g.Wait(); err != nil {
    log.Fatal(err)
}
```

## Реализация на каналах

```go
type Group struct {
    wg   sync.WaitGroup
    once sync.Once
    err  error
    done chan struct{}
}

func NewGroup() *Group {
    return &Group{done: make(chan struct{})}
}

func (g *Group) Go(task func() error) {
    // Проверяем: уже была ошибка?
    select {
    case <-g.done:
        return  // не запускаем
    default:
    }

    g.wg.Add(1)
    go func() {
        defer g.wg.Done()

        // Повторная проверка (между select и go мог закрыться done)
        select {
        case <-g.done:
            return
        default:
        }

        if err := task(); err != nil {
            g.once.Do(func() {    // только один раз:
                g.err = err        // сохранить ошибку
                close(g.done)      // сигнал остальным
            })
        }
    }()
}

func (g *Group) Wait() error {
    g.wg.Wait()
    return g.err
}
```

^eg-impl

## Ключевые моменты

**sync.Once**: ошибка записывается один раз, done закрывается один раз (нельзя закрыть повторно). ^eg-once

**Двойная проверка done**: первая — до создания горутины (быстрый путь). Вторая — внутри горутины (между первой проверкой и запуском мог прийти сигнал). ^eg-double-check

**done-канал**: close(done) → все select'ы с `case <-g.done` срабатывают → новые горутины не запускаются. ^eg-done-channel

## Отличие от errgroup

Реальный `golang.org/x/sync/errgroup`:
- Поддерживает `context.Context` — отменяет контекст при первой ошибке
- Поддерживает лимит конкурентности (`SetLimit`)
- Не блокирует `Go()` — всегда запускает, но контекст отменён

Наша реализация — примитивная версия без контекстов. ^eg-vs-errgroup

## Связь
- [[Done channel]] — done-канал как сигнал об ошибке
- [[interviews/theory/Go/Graceful Shutdown]] — error group для graceful завершения группы задач
- [[Single Flight]] — тоже координация горутин с общим результатом

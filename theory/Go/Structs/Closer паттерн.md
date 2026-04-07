- **Closer** — регистрация cleanup-функций из разных мест, вызов из одного места
- решает проблему: закрытие ресурсов (DB, сеть, воркеры) разбросано по коду
- можно контролировать **порядок** завершения
- **НЕ путать** с finalizer/cleanup (GC) — это разные механизмы
- есть готовые библиотеки (не писать самому)

---

## Проблема

```go
func main() {
    db := openDB()
    server := startServer()
    worker := startWorker()
    cache := initCache()

    // Как закрыть всё при завершении?
    // defer для каждого? А если их 20? А если порядок важен?
}
```

## Примитивная реализация

```go
type Closer struct {
    actions []func() error
}

func (c *Closer) Add(fn func() error) {
    c.actions = append(c.actions, fn)
}

func (c *Closer) Close() error {
    var errs []error
    for _, fn := range c.actions {
        if err := fn(); err != nil {
            errs = append(errs, err)
        }
    }
    // обработать errs...
    return nil
}
```

Closer хранит срез `[]func() error`. `Add` регистрирует cleanup-функцию. `Close` вызывает их по порядку, собирая ошибки. ^closer-impl

## Использование

```go
closer := &Closer{}

db := openDB()
closer.Add(db.Close)           // зарегистрировал из одного места

server := startServer()
closer.Add(server.Shutdown)    // из другого места

worker := startWorker()
closer.Add(worker.Stop)        // из третьего места

// При завершении — одна точка:
defer closer.Close()           // всё закроется по порядку
```

Регистрация — в разных частях кода. Закрытие — из одного места. ^closer-usage

Порядок вызова cleanup-функций соответствует порядку регистрации (FIFO). ^closer-order

## Почему НЕ finalizer/cleanup

```go
// Finalizer (runtime.SetFinalizer) / Cleanup (runtime.AddCleanup):
// - вызывается GC, когда объект больше не достижим
// - может вызваться через 1 итерацию GC, может через 10
// - при завершении программы может НЕ вызваться вообще
// - не контролируешь порядок

// Closer:
// - вызывается ВАШ код, когда ВЫ решите
// - гарантированный порядок
// - гарантированный момент вызова
```

Closer — для детерминированной очистки ресурсов. Finalizer — для «подстраховки» (подробнее в лекции про GC). ^closer-vs-finalizer

Finalizer/Cleanup вызывается GC и **может не вызваться** при завершении программы. Closer вызывается вашим кодом — гарантированно. ^closer-guarantee

## Связь
- [[Functional Options]] — похожий подход: регистрация функций в срезе
- [[Defer механика и порядок]] — defer vs closer: closer даёт больше контроля

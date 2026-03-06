- Done channel — паттерн двух каналов: первый для сигнала отмены (close), второй для подтверждения завершения (closed)
- Контекст даёт только отмену. Done channel даёт отмену + **гарантию завершения**
- Правило: не смешивай контекст + канал. Либо контексты (eventual), либо done channel (с ожиданием)

---

## Проблема

Хочу:
1. Послать горутине/воркеру сигнал "завершайся"
2. **Дождаться**, пока реально завершится

Контекст решает только (1). Для (2) нужен второй канал. ^done-problem

## Реализация для функции

```go
func doWork(closeCh <-chan struct{}) <-chan struct{} {
    closedCh := make(chan struct{})

    go func() {
        defer close(closedCh)  // подтверждение: "я завершился"
        for {
            select {
            case <-closeCh:    // сигнал: "завершайся"
                return
            default:
                // работаем...
                time.Sleep(200 * time.Millisecond)
            }
        }
    }()

    return closedCh
}

// Использование:
closeCh := make(chan struct{})
closedCh := doWork(closeCh)

time.Sleep(2 * time.Second)
close(closeCh)   // шаг 1: посылаем сигнал отмены
<-closedCh       // шаг 2: ждём подтверждения завершения
```

^done-func-impl

## Реализация для структуры (воркер)

```go
type Worker struct {
    closeCh  chan struct{}   // сигнал отмены
    closedCh chan struct{}   // подтверждение
}

func NewWorker() *Worker {
    w := &Worker{
        closeCh:  make(chan struct{}),
        closedCh: make(chan struct{}),
    }
    go w.run()
    return w
}

func (w *Worker) run() {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    defer close(w.closedCh)

    for {
        select {
        case <-w.closeCh:
            return
        case <-ticker.C:
            fmt.Println("working")
        }
    }
}

func (w *Worker) Shutdown() {
    close(w.closeCh)   // "завершайся"
    <-w.closedCh       // ждём
}
```

^done-worker-impl

## Контекст vs Done channel

```
                    Контекст              Done channel
Отмена              ✓                     ✓
Ожидание            ✗                     ✓
завершения
Каналов             0                     2
Когда               eventual termination  guaranteed termination
``` ^done-vs-context

**Не смешивай**: контекст + канал завершения = лишняя сложность. Выбери одно. ^done-no-mix

## Связь
- [[interviews/theory/Go/Graceful Shutdown]] — done channel внутри graceful shutdown
- [[Or-done channel]] — done channel + range по каналу

- Ticket lock = spin lock с fairness: потоки обслуживаются в порядке прихода (FIFO)
- Два счётчика: next (следующий номерок) и owner (кого обслуживаем)
- Поток берёт номерок (atomic add), крутится пока owner != его номер

---

## Проблема spin lock

Обычный spin lock — кто первый схватил CAS, тот вошёл. Нет гарантии порядка. Поток может голодать: другие перехватывают лок раз за разом. ^ticket-spinlock-problem

## Идея — как в очереди с талончиками

```
Вошёл → взял номерок → ждёшь пока табло покажет твой номер
```

```go
type TicketLock struct {
    next  atomic.Uint64  // следующий свободный номерок
    owner atomic.Uint64  // номер текущего владельца
}

func (t *TicketLock) Lock() {
    ticket := t.next.Add(1) - 1  // атомарно берём номерок
    for t.owner.Load() != ticket {
        runtime.Gosched()  // ждём свою очередь
    }
}

func (t *TicketLock) Unlock() {
    t.owner.Add(1)  // вызываем следующий номер
}
```

^ticket-impl

## Почему лучше spin lock

- **FIFO**: потоки входят строго в порядке вызова Lock() ^ticket-fifo
- **Нет голодания**: каждый поток гарантированно дождётся своей очереди ^ticket-no-starvation
- **Простота**: два атомарных счётчика, никаких сложных структур ^ticket-simplicity

## Проблемы ticket lock

- **Всё ещё busy waiting**: потоки крутятся, жгут CPU ^ticket-busy-waiting
- **Cache line bouncing на owner**: все потоки читают `owner` — при каждом `Unlock()` кэш-линия инвалидируется на всех ядрах ^ticket-cache-bouncing
- **Нет пропорциональных backoff**: все потоки одинаково агрессивно проверяют ^ticket-no-backoff

**Оптимизация — proportional backoff:**
Поток знает свой номер и текущий owner. Разница = сколько людей перед ним. Чем больше разница — тем реже проверять. ^ticket-proportional-backoff

## Связь
- [[Spin lock]] — базовый вариант без fairness
- [[True sharing и False sharing]] — cache line bouncing на owner
- [[Примитивный мьютекс]] — park/unpark вместо spinning

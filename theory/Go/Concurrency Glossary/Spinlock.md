- **Spinlock** — блокировка через busy-wait: горутина **крутит CAS** пока не захватит. Не засыпает, не отдаёт CPU ^spin-def
- **Быстро** если КС короткая (наносекунды). **Катастрофа** если длинная — CPU горит впустую ^spin-tradeoff
- Go `sync.Mutex` — **гибрид**: сначала спинит несколько итераций, потом засыпает ^spin-go-hybrid

---

```go
type SpinLock struct{ flag int32 }

func (s *SpinLock) Lock() {
    for !atomic.CompareAndSwapInt32(&s.flag, 0, 1) {
        runtime.Gosched() // отдать управление планировщику
    }
}

func (s *SpinLock) Unlock() {
    atomic.StoreInt32(&s.flag, 0)
}
```
^spin-impl

## Когда использовать

**Да**: КС в несколько наносекунд, context switch дороже ожидания. ^spin-when-yes

**Нет**: в Go почти никогда — планировщик кооперативный, спинлок без `Gosched()` может заблокировать весь P. ^spin-when-no

## Unfair → starvation

Простой спинлок — **нечестный**: кто первый прокрутил CAS — тот и зашёл. Горутина может "голодать" бесконечно. Решение: ticket lock (FIFO). ^spin-unfair

## Связь
- [[Mutex]] — засыпает вместо busy-wait
- [[CAS]] — основа спинлока
- [[Starvation]] — unfair спинлок → голодание

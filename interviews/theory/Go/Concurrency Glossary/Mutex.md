- **Mutex** (Mutual Exclusion) — только **один** владелец критической секции в каждый момент. Lock() блокирует остальных, Unlock() впускает следующего ^mx-def
- Go mutex **нерекурсивный**: повторный Lock из той же горутины → **deadlock** ^mx-not-reentrant
- Go mutex — **гибрид**: сначала спинит, потом засыпает. После 1ms ожидания — **starvation mode** (fair FIFO) ^mx-hybrid

---

```go
var mu sync.Mutex
mu.Lock()
// критическая секция
mu.Unlock()
```
^mx-basic

**RWMutex** — разделяет читателей и писателей. Несколько RLock одновременно, Lock — эксклюзивно. ^mx-rw

## Правила

- Unlock незалоченного мьютекса → **паника** ^mx-unlock-panic
- Lock + Lock в одной горутине → **deadlock** (нерекурсивный) ^mx-double-lock
- `defer mu.Unlock()` — идиома, гарантирует Unlock даже при панике ^mx-defer

## Starvation mode (Go 1.9+)

Если горутина ждёт **>1ms** — мьютекс переключается в fair mode: лок передаётся **напрямую** ожидающей горутине, а не тем, кто только что пришёл. ^mx-starvation-mode

## Стоимость

Горутина блокируется → переходит в **waiting** → context switch → пробуждение. При высоком contention — дорого. ^mx-cost

## Связь
- [[Spinlock]] — busy-wait вместо засыпания
- [[Semaphore]] — mutex = семафор с count=1
- [[Critical Section]] — что именно защищает mutex

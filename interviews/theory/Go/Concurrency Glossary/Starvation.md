- **Starvation** (голодание) — горутина **бесконечно** не получает ресурс, потому что другие постоянно обходят ^stv-def
- Система **работает**, прогресс **есть** — но не у всех. Отличие от deadlock (никто) и livelock (никто, но CPU горит) ^stv-vs-others
- Причины: unfair mutex, приоритеты, бесконечный поток читателей в RWMutex ^stv-causes

---

```
Горутина A: Lock → работа → Unlock → Lock → ...
Горутина B: Lock → работа → Unlock → ...
Горутина C: пытается Lock... Lock... Lock... (никогда не получит)
```
^stv-example

## RWMutex writer starvation

Читатели приходят **непрерывно** → всегда кто-то держит RLock → писатель **никогда** не получит Lock. ^stv-rwmutex

Go решает: если есть ожидающий писатель — **новые** читатели блокируются. ^stv-rwmutex-fix

## sync.Mutex starvation mode

Горутина ждёт **>1ms** → мьютекс переходит в **starvation mode**: лок передаётся напрямую ожидающей горутине (FIFO), а не свежепришедшей. ^stv-go-mutex

## Связь
- [[WORK-BASE/interviews/theory/Go/Concurrency Glossary/Deadlock]] — никто не продвигается
- [[Livelock]] — никто, но CPU горит
- [[Mutex]] — starvation mode в Go

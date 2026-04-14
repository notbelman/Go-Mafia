- **CAS** (Compare-And-Swap) — атомарная инструкция CPU: "если `*addr == old` → записать `new`, вернуть true". Иначе false ^cas-def
- **Основа всех lock-free** алгоритмов. CAS-луп = load → compute → CAS → retry ^cas-loop
- Гарантия прогресса: если мой CAS провалился — **чей-то CAS прошёл**. Система всегда движется ^cas-progress

---

```go
// Go:
atomic.CompareAndSwapInt64(&value, old, new)
```
^cas-go

## CAS-луп (стандартный паттерн)

```go
for {
    old := atomic.LoadInt64(&counter)
    new := old + 1
    if atomic.CompareAndSwapInt64(&counter, old, new) {
        break // успех
    }
    // кто-то встроился → повторяем
}
```
^cas-loop-example

## Проблемы

**ABA**: адрес тот же, данные другие. Аллокатор переиспользовал память. В Go не возникает (GC), актуально для C/C++. ^cas-aba

**Высокий contention**: много горутин → много CAS-неудач → CPU крутится впустую. ^cas-contention

## Связь
- [[Lock-free]] — CAS = основа lock-free алгоритмов
- [[Spinlock]] — спинлок = CAS на флаге
- [[ABA-проблема]] — подробнее про ABA

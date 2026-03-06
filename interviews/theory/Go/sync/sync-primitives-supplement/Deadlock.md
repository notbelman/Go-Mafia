- взаимная блокировка: горутины ждут ресурсы друг друга, ни одна не может продолжить ^deadlock-definition
- runtime Go обнаруживает deadlock **только если ВСЕ горутины в waiting**. Одна работающая — и сотня в deadlock не обнаружится ^deadlock-runtime-detection
- минимум мьютексов: концептуально 2, но в Go достаточно 1 (повторный Lock = вечная блокировка) ^deadlock-one-mutex

---

## Несогласованный порядок захвата

```go
func normalize(mu1, mu2 *sync.Mutex) {
    mu1.Lock()
    mu2.Lock()
    // работа
    mu2.Unlock()
    mu1.Unlock()
}

// 500 горутин: normalize(muA, muB)
// 500 горутин: normalize(muB, muA)  ← порядок нарушен!
```

G1 захватила muA, ждёт muB. G2 захватила muB, ждёт muA. Всё, мёртвый груз — только рестарт приложения. ^deadlock-lock-order

**Решение:** всегда одинаковый порядок захвата. Когда захватываете больше одного мьютекса — должна зажечься лампочка: «порядок согласован?» ^deadlock-lock-order-solution

## Runtime Go НЕ всегда видит deadlock

```go
go func() { for { time.Sleep(time.Second) } }()  // одна живая горутина

var mu sync.Mutex
mu.Lock()
mu.Lock()  // deadlock, но runtime НЕ скажет!
```

Runtime сообщает "all goroutines are asleep" только когда **все** горутины в waiting. Если хоть одна работает (а в реальных сервисах их тысячи) — deadlock пройдёт незамеченным. ^deadlock-hidden

## Deadlock одним мьютексом

```go
var mu sync.Mutex
mu.Lock()
mu.Lock()  // вечная блокировка
```

Go мьютекс **не хранит ID владельца** (не reentrant). Второй Lock видит «занято» и блокируется навечно. Никто не разбудит. ^deadlock-not-reentrant

## Unlock из другой горутины — ок

```go
mu.Lock()
go func() {
    mu.Unlock()  // работает! мьютекс не знает, кто его брал
}()
```

^deadlock-unlock-other-goroutine

## Unlock незалоченного мьютекса — паника

```go
var mu sync.Mutex
mu.Unlock()  // panic: sync: unlock of unlocked mutex
```

^deadlock-unlock-panic

## Связь
- [[Структура]] — мьютекс не хранит goroutine ID
- [[Livelock и Starvation]] — другие проблемы конкурентности
- [[Два режима (с Go 1.9)]] — starvation mode как защита от голодания

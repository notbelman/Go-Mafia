- livelock: горутины **не заблокированы**, что-то делают, но бесполезно — нет гарантии прогресса ^livelock-definition
- starvation: горутина не может получить ресурс из-за «жадных» конкурентов ^starvation-definition
- deadlock = стоят, livelock = бегают на месте, starvation = одного не пускают ^three-problems-comparison

---

## Livelock

```go
func routine1(mu1, mu2 *sync.Mutex) {
    for {
        mu1.Lock()
        if !mu2.TryLock() {
            mu1.Unlock()
            runtime.Gosched()  // «уступи другому»
            continue           // и попробуй снова
        }
        // работа
        mu2.Unlock()
        mu1.Unlock()
        return
    }
}

func routine2(mu1, mu2 *sync.Mutex) {
    for {
        mu2.Lock()
        if !mu1.TryLock() {
            mu2.Unlock()
            runtime.Gosched()
            continue
        }
        // работа
        mu1.Unlock()
        mu2.Unlock()
        return
    }
}
```

Обе горутины бесконечно: захватить → не смочь → отпустить → повторить. Ядра греются, процесс крутится, но полезной работы = 0. ^livelock-code

Встречается при работе с lock-free структурами и атомиками. ^livelock-where

## Starvation (голодание)

```go
// Жадный воркер: захватил — спит 3ns — отпустил
go greedyWorker(mu)   // Lock → sleep(3ns) → Unlock → repeat

// Вежливый воркер: 3 раза по 1ns
go politeWorker(mu)   // Lock → sleep(1ns) → Unlock (×3) → repeat
```

Жадный воркер держит мьютекс долго → вежливый голодает, выполняет гораздо меньше работы. ^starvation-example

Может быть не только с памятью: коннекшены, файловые дескрипторы, любые разделяемые ресурсы. ^starvation-resources

## Сравнение

| Проблема | Горутины | Ресурсы | Прогресс |
|:---------|:---------|:--------|:---------|
| Deadlock | заблокированы | заняты навечно | нет |
| Livelock | активны | постоянно берут/отдают | нет |
| Starvation | активны | достаются не всем | частичный |

^comparison-table

## Связь
- [[WORK-BASE/interviews/theory/Go/sync/sync-primitives-supplement/Deadlock]] — взаимная блокировка
- [[Два режима (с Go 1.9)]] — starvation mode мьютекса решает голодание горутин в очереди
- [[sync.Mutex.TryLock (Go 1.18+)]] — TryLock в цикле может приводить к livelock

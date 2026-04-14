## Проблема: без синхронизации нет гарантий

Компилятор и процессор могут переупорядочить операции с памятью (reordering). Значение записанное в одной горутине может быть не видно в другой — застряло в кеше CPU или инструкции переставились. ^mm-reordering-problem

```go
var x, y int

// Горутина 1
go func() {
    x = 1
    y = 2
}()

// Горутина 2
go func() {
    if y == 2 {
        fmt.Println(x) // может напечатать 0! запись x могла не дойти
    }
}()
```
^mm-reordering-example

## Happens-before

Go Memory Model определяет: запись в одной горутине **гарантированно видна** в другой только если между ними есть happens-before связь. Без неё — поведение не определено. ^mm-hb-definition

Что создаёт happens-before: ^mm-hb-sources
- `sync.Mutex` — Lock/Unlock
- `sync.RWMutex` — RLock/RUnlock
- `sync/atomic` — Store/Load
- Каналы — send happens-before receive
- `sync.Once` — Do
- `sync.WaitGroup` — Add/Done/Wait

## Memory Barriers (Memory Fences)

Под капотом все эти механизмы вставляют **memory barrier** — специальную инструкцию процессора, которая запрещает переупорядочивание через эту точку. ^mm-barrier-definition

```
x86 инструкции:
MFENCE — полный барьер (запрещает любое переупорядочивание)
LFENCE — барьер на чтение
SFENCE — барьер на записи
```
^mm-barrier-instructions

Mutex.Lock() под капотом делает atomic операцию + memory barrier → все записи до Unlock() гарантированно видны после следующего Lock(). ^mm-mutex-barrier

## Итого

| Без синхронизации            | С синхронизацией              |
| :--------------------------- | :---------------------------- |
| Компилятор переставляет код  | Барьер запрещает reordering   |
| CPU кеширует значения        | Барьер сбрасывает кеш         |
| Другая горутина видит старое | Гарантия happens-before       |
^mm-summary-table

## Связь
- [[Happens-Before в Go]] — таблица happens-before
- [[Memory Ordering]] — CPU/compiler reordering
- [[Race Detector]] — обнаружение нарушений

- 4 фазы: Sweep Termination (STW) → Mark (concurrent) → Mark Termination (STW) → Sweep (concurrent)
- STW паузы: ~10-30µs + ~60-90µs, основная работа concurrent
- триггер: heap goal, 2 мин без GC, runtime.GC()

---
[[GC Flashcards - overview]]
## Триггер запуска

Heap вырос до `Live heap + (Live heap + stacks + globals) × GOGC/100` — момент запуска выбирает **[[GC Pacer]]**, параметры задаются через **[[GOGC и GOMEMLIMIT]]**. ^gc-trigger-formula

Также запускается если прошло >2 минут без GC (sysmon) или вызван `runtime.GC()`. ^gc-trigger-other

## Фаза 1: Sweep Termination (STW, ~10-30µs)

Останавливаем все горутины. Дочищаем sweep предыдущего цикла (если не закончили). Включаем **[[Write barrier]]**. Возобновляем горутины. ^phase-sweep-term

## Фаза 2: Mark (concurrent, ~25% CPU)

Работает **одновременно с программой**, несколькими потоками параллельно. ^phase-mark-concurrent

Объекты не помечены (white по умолчанию). От root objects (стеки горутин + глобальные + runtime) обходим граф, помечаем живые — это **[[Tri-color marking]]**. ^phase-mark-roots

Программа продолжает работать и меняет указатели — **[[Write barrier]]** следит, чтобы GC не потерял живой объект. ^phase-mark-wb

Если горутина аллоцирует быстрее, чем GC маркирует — **[[Mark Assist]]** заставляет её помогать GC. ^phase-mark-assist

Объекты с **[[Finalizers]]** не удаляются — ставятся в очередь, будут освобождены в следующем цикле. ^phase-mark-finalizers

## Фаза 3: Mark Termination (STW, ~60-90µs)

Останавливаем горутины. Дочищаем очередь серых объектов. Убеждаемся что серых нет. Выключаем **[[Write barrier]]**. Собираем статистику для следующего цикла. Возобновляем горутины. ^phase-mark-term

## Фаза 4: Sweep (concurrent + lazy)

Работает в фоне параллельно с программой. Освобождает white объекты — память возвращается **[[Пример аллокации]]**. ^phase-sweep-bg

Часть sweep происходит прямо при новых аллокациях (lazy). Scavenger возвращает совсем ненужное ОС. ^phase-sweep-lazy

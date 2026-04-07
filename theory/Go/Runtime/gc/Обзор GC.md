- **4 фазы:** Sweep Termination (**STW** ~10-30µs) → Mark (**concurrent**, ~25% CPU) → Mark Termination (**STW** ~60-90µs) → Sweep (**concurrent** + lazy). Основная работа concurrent, STW паузы короткие
- **Триггер:** heap вырос до `Live heap + (Live heap + stacks + globals) × GOGC/100` (выбирает [[GC Pacer]]). Также: >2 минут без GC (sysmon) или `runtime.GC()`
- **Фаза Mark:** от root objects (стеки горутин + глобальные + runtime) обходит граф → [[Tri-color marking]]. [[Write barrier]] следит за корректностью (мутатор меняет указатели). [[Mark Assist]] заставляет горутину помогать если аллоцирует быстрее чем GC маркирует
- **Go 1.26+ ([[Green Tea GC (new)]]):** unit работы в Mark фазе сменился с объекта на страницу 8 KiB. Sequential scans вместо depth-first jumps → 10–40% снижение GC CPU overhead
- **Финалайзеры:** объекты с [[Finalizers]] не удаляются в текущем цикле — ставятся в очередь, удаляются в следующем (+1 цикл)
- **Sweep:** освобождает white (непомеченные) объекты concurrent в фоне. Часть sweep происходит lazy (при новых аллокациях). Scavenger возвращает память ОС

---

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

> [!warning] Устарело (до Go 1.26 / до Green Tea)
> До Go 1.26 unit работы = **отдельный объект**, обход depth-first (LIFO stack).
> Начиная с Go 1.26 используется [[Green Tea GC (new)]]: unit = страница 8 KiB, FIFO queue.

**Go 1.26+: [[Green Tea GC (new)]]** — фаза Mark работает со **страницами** (8 KiB), а не отдельными объектами. Последовательные проходы по странице в memory order вместо scattered depth-first jumps. Снижение GC CPU overhead: 10–40%. ^phase-mark-greentea

## Фаза 3: Mark Termination (STW, ~60-90µs)

Останавливаем горутины. Дочищаем очередь серых объектов. Убеждаемся что серых нет. Выключаем **[[Write barrier]]**. Собираем статистику для следующего цикла. Возобновляем горутины. ^phase-mark-term

## Фаза 4: Sweep (concurrent + lazy)

Работает в фоне параллельно с программой. Освобождает white объекты — память возвращается в аллокатор. ^phase-sweep-bg

Часть sweep происходит прямо при новых аллокациях (lazy). Scavenger возвращает совсем ненужное ОС. ^phase-sweep-lazy

## Связь
- [[GC Pacer]] — алгоритм выбора момента запуска
- [[GOGC и GOMEMLIMIT]] — параметры управления GC
- [[Tri-color marking]] — алгоритм маркировки в фазе Mark
- [[Write barrier]] — корректность concurrent Mark
- [[Mark Assist]] — скорость: горутины помогают GC
- [[Finalizers]] — объекты с финалайзерами живут +1 цикл
- [[Green Tea GC (new)]] — Go 1.26+: новый алгоритм Mark фазы

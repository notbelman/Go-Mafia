## Memory в Go

### Стек vs Куча

#### [[Стек vs Куча]]
Стек — автоматический, куча — GC управляет, trade-offs

#### [[Стек]]
Структура горутинного стека, начальный размер 2-8KB

#### [[Стек пример]]
Пример стекового фрейма вызова функции

#### [[Stack growth]]
Contiguous stack, копирование при росте, stack shrinking

#### [[Почему стек ОС-треда 1-8 MB]]
Фиксированный стек OS-треда vs динамический горутинный

#### [[Направление роста стека и кучи]]
Стек растёт вниз, куча вверх, guard pages

#### [[Куча что это]]
Heap layout в Go, арены, spans, pages

---

### Аллокатор

#### [[TCMalloc - почему Go так делает]]
Google TCMalloc как основа Go аллокатора

#### [[Алгоритмы аллокации (база)]]
Bump pointer, free list, segregated fits

#### [[Три уровня аллокатора (mcache - mcentral - mheap)]]
mcache (per-P, no lock) → mcentral (per-class) → mheap (global)

#### [[Классы размеров (size classes)]]
67 size classes, ~0-32KB малые объекты, крупные через mheap

#### [[Организация памяти кучи (арены - страницы - спаны)]]
Arena 64MB, page 8KB, span = набор pages одного size class

#### [[Пример аллокации]]
Полный путь аллокации объекта через mcache → mcentral → mheap

#### [[Арены (experimental)]]
arena пакет (Go 1.20 experimental), bulk allocation

---

### Escape Analysis

#### [[Escape analysis - что это и зачем]]
Статический анализ: heap или stack? Компилятор решает

#### [[Что вызывает escape]]
Interface boxing, closures, слишком большие объекты, &local

#### [[Inlining и escape]]
Inlining уменьшает escape, границы функций = escape boundary

#### [[Practical приёмы уменьшения аллокаций]]
sync.Pool, pre-allocation, value semantics, избегать interface boxing

---

### Продвинутое

#### [[Выравнивание (alignment)]]
Alignment требования CPU, false sharing, padding

#### [[Atomic и выравнивание]]
64-bit atomic на 32-bit платформах требуют выравнивания

#### [[sync.Pool]]
Pool для переиспользования объектов, GC и Pool interaction

#### [[Memory model subfolder]]
→ см. папку `memory model/`

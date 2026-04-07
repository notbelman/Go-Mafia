- **Write barrier** — при записи указателя в heap → уведомить GC. Нужен потому что в concurrent режиме мутатор меняет указатели пока GC маркирует. Без barrier → «чёрный → белый» → GC удалит живой объект (lost object)
- **Insertion barrier (Dijkstra):** `A.field = C` → новый ребёнок C красится **серым**. **Минус:** стек не покрыт → в Mark Termination нужен полный **rescan стеков** (долгий STW)
- **Deletion barrier (Yuasa):** `A.field = C` → старый ребёнок B красится **серым**. **Минус:** мусор живёт **+1 цикл GC** (старый ребёнок серый → не удалится в этом цикле)
- **Hybrid barrier (Go 1.8+):** и старый и новый ребёнок → серые. Стек **не покрыт** (дорого) — стеки сканируются целиком один раз при приостановке горутины. **Rescan не нужен → паузы короче**
- **Overhead:** ~2 доп. инструкции на запись указателя. Fast path: проверка `writeBarrier.enabled` (почти бесплатно когда GC не активен). Общий ~10-20% на pointer-heavy код
- **Активен только в Mark phase.** Включение/выключение требует STW (~10-30µs). Corner case: очередь серых не пустеет → после лимита новые объекты сразу чёрные, мусор доживёт до следующего цикла

---

**Что такое write barrier**

**Общий принцип:** при записи указателя в heap → уведомить GC. Нужен потому что в concurrent режиме программа меняет указатели пока GC маркирует. Без barrier GC может потерять живой объект (lost object). ^wb-what

**Insertion barrier (Dijkstra):**
```
A.field = C   // записали новый указатель A → C
              // новый ребёнок C → красится серым
```

^72434e

Появился **новый ребёнок** → красим его **серым**, чтобы GC не пропустил. ^wb-insertion-mechanism

**Минус insertion barrier:** стек не покрыт → в Mark Termination нужен полный rescan стеков (долгий STW). ^wb-insertion-minus

**Deletion barrier (Yuasa):**
```
// было: A.field = B (B — ребёнок A)
A.field = C   // перезаписали: B отвязан от A
              // старый ребёнок B → красится серым
```

^1f3190

**Ребёнок потерял родителя** → красим его **серым**, чтобы GC не потерял B и его потомков. ^wb-deletion-mechanism

**Минус deletion barrier:** мусор живёт +1 цикл GC (старый ребёнок серый → не удалится в этом цикле). ^wb-deletion-minus

**Hybrid (Go 1.8+):**
```
// было: A.field = B
A.field = C   // и старый ребёнок B → серый
              // и новый ребёнок C → серый
```

^77f78a

Стек НЕ покрыт barrier (дорого). Стеки сканируются целиком один раз при приостановке горутины. ^wb-hybrid-stack

Rescan стеков не нужен → паузы короче. ^wb-hybrid-benefit

**Когда активен в Go**

Mark phase: write barrier ON. Sweep phase: write barrier OFF. ^wb-phases

Включение/выключение требует STW (~10-30µs). ^wb-when-active

**Корнер-кейс: серые не заканчиваются.** Мутаторы плодят объекты → все красятся серым → очередь не пустеет. ^wb-corner-case-problem

Решение: после лимита красить новые объекты сразу чёрными. Мусор доживёт до следующего цикла, но фаза Mark завершится. ^wb-corner-case-solution

**Overhead**

~2 доп. инструкции на запись указателя. ^wb-overhead-instructions

Fast path: проверка `writeBarrier.enabled` (почти бесплатно когда GC не активен). ^wb-overhead-fastpath

Общий overhead: ~10-20% на pointer-heavy код. ^wb-overhead-total

**Эволюция Go**

Go 1.5: insertion barrier → STW rescan стеков. ^wb-go15

Go 1.8: hybrid barrier → нет rescan → короче паузы. ^wb-go18

## Связь
- [[Tri-color marking]] — алгоритм маркировки, который write barrier поддерживает
- [[Обзор GC]] — когда barrier включён/выключен (фазы 1 и 3)
- [[Mark Assist]] — другой механизм поддержки concurrent GC (про скорость, не корректность)

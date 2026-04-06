- **Go 1.26 default.** Принцип: работать со **страницами 8 KiB**, а не отдельными объектами → sequential scans в memory order вместо scattered depth-first jumps → лучше cache locality
- **Проблемы старого GC (graph flood):** cache miss на каждый переход по pointer (объекты не обязаны быть рядом); NUMA штрафы; снижение memory bandwidth per core; contention на shared work list; vector hardware неприменимо (объекты разных размеров)
- **Два бита на объект-слот страницы:** `seen` (найден pointer на объект) + `scanned` (объект уже просканирован в этом проходе). Work list = **страницы** (FIFO queue). FIFO: даёт накопиться нескольким seen objects на странице до её обработки → дольше sequential scans
- **Алгоритм:** pointer → `seen[O]=1` в metadata страницы P → P в FIFO если не там → обработка P: scan всех `(seen=1 AND scanned=0)` в memory order → для каждого pointer повтор. P может попасть в work list повторно (новые seen bits)
- **10–40% снижение GC CPU overhead.** Modal ~10%. Пример: 10% CPU на GC → экономия 1–4% total. +10% на Intel Ice Lake / AMD Zen 4+ (AVX-512), см. [[Green Tea - Vector acceleration (new)]]
- **Edge case:** 1 объект на страницу → накоплений нет → special fast path. Достаточно **2% заполнения страницы** чтобы выиграть у graph flood
- **Availability:** Go 1.25 opt-in (`GOEXPERIMENT=greenteagc`). Go 1.26 default, opt-out (`GOEXPERIMENT=nogreenteagc`). Go 1.27: opt-out удалён

---

## Проблема старого GC

До Green Tea фаза Mark работала как **graph flood**: depth-first обход, каждый объект — отдельная запись в LIFO work list.

```
Старый GC (graph flood, LIFO):
  A ──ptr──▶ B ──ptr──▶ C
             │
             └──ptr──▶ D ──ptr──▶ E

Порядок: A → B → C, backtrack → D → E
Каждая стрелка = вероятный cache miss (объекты не рядом в памяти)
7 отдельных scans по всему heap
```

**Cache misses.** Два связанных pointer'ом объекта не обязаны быть рядом в памяти. Каждый переход → вероятный промах кэша → ожидание main memory (до 100× медленнее кэша). Каждый шаг зависит от предыдущего — CPU не может перекрыть промахи параллельными запросами. ^gt-cache-miss

**NUMA.** Современные CPU: память привязана к подмножеству ядер. Доступ «чужого» ядра дороже. Scattered jumps по heap усиливают NUMA-штрафы. ^gt-numa

**Снижение memory bandwidth per core.** Ядер становится больше, но bandwidth на ядро снижается. Scattered доступы создают конкуренцию за bandwidth. ^gt-bandwidth

**Contention на shared work list.** Mark фаза параллельна. Единый work list из объектов → shared state → lock contention. ^gt-contention

**Vector hardware неприменимо.** Объекты в куче разных размеров и типов, работа нерегулярная → невозможно использовать SIMD/AVX. ^gt-no-vector

> [!quote] Austin Clements (Go team)
> "A microarchitectural disaster" — характеристика depth-first graph flood на современном железе.

## Идея

**Работать со страницами, а не объектами.** ^gt-idea

**Страница** в контексте Go runtime — 8 KiB выровненный блок памяти, содержащий объекты **одного size class**. Все объекты на странице одинакового размера. ^gt-page-def

```
Страница 8 KiB (все объекты одного size class):
┌────┬────┬────┬────┬────┬────┬────┬────┐
│obj │obj │obj │obj │obj │obj │obj │obj │  ...
└────┴────┴────┴────┴────┴────┴────┴────┘

Metadata (per-page, per-object-slot):
  seen bits:    0 1 0 1 1 0 0 1  ...
  scanned bits: 0 0 0 1 0 0 0 0  ...
```

**`seen` bit** — указатель на объект был обнаружен (кто-то сослался). ^gt-seen-bit

**`scanned` bit** — объект уже просканирован в текущем проходе (его pointers обработаны). ^gt-scanned-bit

Work list хранит **страницы** вместо объектов → размер work list значительно меньше → меньше contention. ^gt-worklist-pages

## Алгоритм

```
1. Нашли pointer на объект O
   → seen[O] = 1 в metadata страницы P
   → P не в FIFO work list → добавить в конец
          ↓
2. Взяли страницу P из начала FIFO очереди
   → найти все объекты: seen=1 AND scanned=0
   → просканировать в memory order (left-to-right)
   → для каждого найденного pointer → шаг 1
   → scanned=1 для просканированных объектов
          ↓
3. P может снова попасть в work list
   → пока обрабатывали P, другие потоки выставили новые seen bits
   → P переобрабатывается: снова (seen AND NOT scanned)
          ↓
4. Work list пуст → sweep
```

^gt-algorithm

**Почему FIFO, а не LIFO?** LIFO (depth-first) немедленно уходит вглубь — страница обрабатывается с 1 объектом. FIFO (breadth-first) даёт время накопиться нескольким `seen` objects на странице до её обработки → longer sequential scans → лучше cache locality. ^gt-fifo-why

```
Green Tea (FIFO):
  Страница A: 4 объекта накопилось → один длинный scan
  Страница B: 3 объекта накопилось → один длинный scan

  Итого: 4 прохода вместо 7 (у graph flood)
  Чем больше heap — тем сильнее эффект
```

^gt-result

## Сравнение: старый GC vs Green Tea

| Аспект | Старый GC (до Go 1.26) | Green Tea (Go 1.26+) |
|---|---|---|
| Unit работы | объект | страница 8 KiB |
| Work list | LIFO (stack) | FIFO (queue) |
| Порядок обхода | depth-first | memory order (sequential) |
| Cache locality | плохая (scattered) | хорошая (sequential) |
| Vector instructions | невозможно | AVX-512 (Ice Lake/Zen 4+) |
| Contention | высокое | ниже |

^gt-comparison

## Edge cases

**Один объект на страницу.** Крупные объекты (каждый занимает целую страницу) → накопления нет → может быть чуть медленнее graph flood. В реализации есть special fast path для single-object pages. ^gt-single-object

**Порог эффективности.** Достаточно сканировать **2% страницы за проход** чтобы выиграть у graph flood. Даже умеренно заполненные страницы дают выигрыш. ^gt-threshold

## Performance

10–40% снижение GC CPU overhead в зависимости от workload. ^gt-perf-range

Modal improvement: **~10%**. ^gt-perf-modal

Пример: программа тратит 10% CPU на GC → экономия **1–4% total CPU**. ^gt-perf-example

Дополнительно **+10%** снижения GC CPU на платформах с AVX-512 (Intel Ice Lake, AMD Zen 4+) — см. [[Green Tea - Vector acceleration (new)]]. ^gt-perf-avx

Уже в production у Google с подтверждёнными результатами. ^gt-perf-google

## Availability

```
Go 1.25 (окт 2025):  GOEXPERIMENT=greenteagc    → opt-in, без vector
Go 1.26 (фев 2026):  default                    → opt-out: GOEXPERIMENT=nogreenteagc
Go 1.27 (план):      nogreenteagc удалён         → старого GC больше нет как опции
```

^gt-availability

## Связь
- [[Обзор GC]] — фазы GC, где работает Green Tea (фаза 2: Mark)
- [[Tri-color marking]] — алгоритм маркировки; Green Tea меняет unit работы, инварианты те же
- [[Green Tea - Vector acceleration (new)]] — AVX-512 ускорение сканирования
- [[Поколения (generational GC)]] — Green Tea ≠ generational; другое направление оптимизации

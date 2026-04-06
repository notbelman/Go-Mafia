# Версии Go — ключевые изменения

TL;DR аналог [go.dev/doc/go1.X](https://go.dev/doc/go1) — только важное для разработчика.

---

## Go 1.26 (февраль 2026)

### Runtime / GC
- **Green Tea GC по умолчанию** — 10-40% снижение GC CPU; page-based marking (8 KiB страницы), FIFO work list, лучше cache locality → [[Green Tea GC (new)]]
- **Green Tea Vector acceleration** — AVX-512 на Intel Ice Lake / AMD Zen 4+: дополнительно ~10% GC CPU → [[Green Tea - Vector acceleration (new)]]
- **Heap base address randomization** — ASLR для Go heap: рандомизация базового адреса кучи (безопасность, усложняет эксплуатацию memory corruption)
- **Faster cgo calls** — ~30% ускорение вызовов cgo за счёт оптимизации перехода Go↔C

### Горутины
- **Goroutine leak profile** — `GOEXPERIMENT=goroutineleakprofile`: новый pprof-профиль `/debug/pprof/goroutineleak`; GC находит горутины заблокированные на primitives (channel/mutex/cond), недостижимых от любой runnable горутины → [[Нюансы горутин]]

---

## Go 1.25 (октябрь 2025)

### Runtime
- **Container-aware GOMAXPROCS** — Go 1.25+ автоматически читает cgroup CPU bandwidth limit на Linux (= CPU limit в K8s); `uber-go/automaxprocs` больше не нужен; периодически обновляется если лимит изменился; отключить: `GODEBUG=containermaxprocs=0` → [[P (Processor)]]
- **Green Tea GC opt-in** — `GOEXPERIMENT=greenteagc` для раннего включения (стал default в 1.26) → [[Green Tea GC (new)]]
- **Trace flight recorder** — `runtime/trace.FlightRecorder`: буферизированная трассировка в памяти, dump по требованию (production профилирование без постоянной записи)

### GC / Memory
- **runtime.AddCleanup** — замена `runtime.SetFinalizer`: несколько cleanups на одном объекте, не привязан к типу, безопаснее → [[Finalizers]]

---

## Go 1.24 (февраль 2025)

### Runtime / Map
- **Swiss Tables для map** — новая реализация хэш-таблицы (Google Swiss Tables): лучше cache locality за счёт linear probing + SIMD-friendly metadata; быстрее lookup и итерация

### Memory
- **weak.Pointer** — новый пакет `weak`: слабые указатели не предотвращают GC объекта; полезны для кэшей и интернирования без удержания памяти

---

## Go 1.23 (август 2024)

### Язык
- **range over functions** — стабильный синтаксис push-итераторов: `for v := range f` где `f` имеет тип `func(yield func(V) bool)`; устранил необходимость в кастомных паттернах итерации → [[Iterators]]

### Стандартная библиотека
- **Timer/Ticker: Stop/Reset без drain** — канал таймера теперь ёмкость 1 → effectively unbuffered для новых случаев; `Stop`/`Reset` дренируют канал автоматически; старый паттерн `select { case <-t.C: default: }` перед `Reset` больше не нужен (и может вызвать race condition)

---

## Go 1.22 (февраль 2024)

### Язык
- **Loop variable per iteration** — каждая итерация `for` создаёт новую переменную; классическая ловушка `go func() { use(i) }()` в цикле устранена на уровне компилятора → [[Нюансы горутин]]
- **range over integers** — `for i := range 10` без временной переменной (0..9)

---

## Go 1.21 (август 2023)

### Стандартная библиотека
- **log/slog** — структурированное логирование в stdlib: `slog.Info("msg", "key", val)`, backend-агностичный через `slog.Handler`
- **slices, maps, cmp** — generic-пакеты: `slices.Sort`, `slices.Contains`, `maps.Keys`, `maps.Clone`, `cmp.Compare`, `cmp.Ordered`
- **sync.OnceFunc / sync.OnceValue / sync.OnceValues** — удобные обёртки над `sync.Once` для функций с результатом

### Runtime / GC
- **40% снижение tail latency GC** — оптимизации планировщика и GC, улучшение хвостовых задержек P99+

---

## Связь
- [[Green Tea GC (new)]] — детали Go 1.26 GC
- [[Green Tea - Vector acceleration (new)]] — AVX-512 ускорение
- [[P (Processor)]] — container-aware GOMAXPROCS
- [[Нюансы горутин]] — goroutine leak profile, loop variable fix
- [[Finalizers]] — runtime.AddCleanup
- [[Iterators]] — range over functions
- [[Обзор GC]] — полный цикл GC в Go

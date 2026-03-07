- **GOGC**: % роста heap до следующего GC (default 100 → next GC = prev × 2). Трейдофф:  GOGC↓ = чаще GC, меньше памяти, больше CPU. GOGC↑ = реже GC, больше памяти, меньше CPU.
- **GOMEMLIMIT**: soft-лимит на **всю** память, runtime сам уменьшает GOGC. Защита: CPU GC **≤ 50%** при  Если не справляется — позволяет превысить лимит (soft, не hard). Иначе deadlock.
- **death spiral:** live heap ≈ лимит
	  → GC постоянно работает
	  → CPU 100% на GC (но ≤50% с GOMEMLIMIT)
	  → программа не обрабатывает запросы
	  → запросы копятся → нужно ещё больше памяти → ...

---
[[GC Flashcards - gogc_gomemlimit]]

Два способа управления GC: частота (GOGC) и лимит памяти (GOMEMLIMIT).

## GOGC — частота сборки

```
Процент роста heap до следующего GC. Default: 100

next GC = live heap × (1 + GOGC/100)

GOGC=100, live heap=100MB → GC при 200MB  (×2)
GOGC=50,  live heap=100MB → GC при 150MB  (×1.5)
GOGC=200, live heap=100MB → GC при 300MB  (×3)
```
^gogc-formula

Трейдофф: GOGC↓ = чаще GC, меньше памяти, больше CPU. GOGC↑ = реже GC, больше памяти, меньше CPU. ^gogc-tradeoff

**Проблема ручного подбора:** значение зависит от нагрузки. Сегодня 50, завтра нужно 80, послезавтра 30. Вечная ручная подстройка. ^gogc-manual-problem

**Пример OOM:** VM с лимитом 10GB, GOGC=100. После GC heap = 5.1GB. Следующий GC при 5.1 × 2 = 10.2GB > лимит VM → OOM-killer убьёт процесс до срабатывания GC. ^gogc-oom

## Ballast-хак

```go
var ballast = make([]byte, 2<<30)  // 2GB
```

^3bfe76

Аллокация большого массива для поднятия порога GC. Работает из-за **lazy allocation** ОС: пока не пишем в память, RSS не растёт (только VSS). С появлением GOMEMLIMIT в 99% случаев не нужен. 
^ballast-hack

## GOMEMLIMIT (Go 1.19) — soft memory limit

**GOMEMLIMIT (Go 1.19) — soft memory limit**

Учитывает **всю** память (heap + runtime metadata), не только кучу. НЕ включает: memory-mapped files, cgo memory. ^gomemlimit-what

Runtime динамически крутит GOGC при приближении к лимиту. ^gomemlimit-dynamic

**Защита от death spiral:** CPU на GC ≤ 50%. Если не справляется — позволяет превысить лимит (soft, не hard). Иначе deadlock. ^gomemlimit-soft

```
# Рекомендация в контейнерах
GOMEMLIMIT = container_limit * 0.9

# Обычный режим
GOGC=100 GOMEMLIMIT=2GiB

# Максимум памяти, минимум GC
GOGC=off GOMEMLIMIT=4GiB → GC только при приближении к лимиту

# Агрессивный GC
GOGC=50 GOMEMLIMIT=512MiB
```
^gomemlimit-configs

**Программно:**
```go
debug.SetGCPercent(50)         // GOGC=50
debug.SetGCPercent(-1)         // выключить GC
debug.SetMemoryLimit(1 << 30)  // 1GiB
```
^gogc-programmatic

## Death spiral

```
live heap ≈ лимит
  → GC постоянно работает
  → CPU 100% на GC (но ≤50% с GOMEMLIMIT)
  → программа не обрабатывает запросы
  → запросы копятся → нужно ещё больше памяти → ...
```

^0f0da8

Сложнее всего обнаружить: приложение работает, CPU загружен, но throughput нулевой. ^death-spiral-detect

С GOGC легко попасть при ручном подборе. С GOMEMLIMIT — защита через ограничение 50% CPU. ^death-spiral

## Связь
- [[GC Pacer]] — алгоритм выбора момента запуска
- [[Mark Assist]] — горутины помогают GC при высокой нагрузке
- [[Обзор GC]] — фазы и триггеры GC
- [[Lazy allocation и RSS vs VSS]] — почему ballast не ест физическую память

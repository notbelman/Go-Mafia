- **Data race** = нарушение **memory model** (UB). **Race condition** = нарушение **логики** (неверный результат) ^drrc-key
- Они **независимы**: DR может быть без RC, RC может быть без DR ^drrc-independent
- `-race` ловит **только data race**. Race condition — не ловит никакой инструмент ^drrc-detection

---

| | Data Race | Race Condition |
|:--|:--|:--|
| **Что** | Одновременный доступ к памяти | Логическая ошибка порядка |
| **Условие** | 2+ горутины, 1+ запись, нет sync | Результат зависит от тайминга |
| **Последствие** | **Undefined Behavior** | Неверный результат |
| **-race ловит** | ✅ | ❌ |
| **Пример** | `go func(){ n++ }(); print(n)` | check-then-act с mutex |
^drrc-table

## DR без RC

Две горутины пишут **одно и то же значение** — data race есть (нет sync), но результат всегда правильный. ^drrc-dr-without-rc

## RC без DR

Все операции через mutex/channel (data race нет), но check-then-act ломает логику. ^drrc-rc-without-dr

## Связь
- [[Data Race]] — определение
- [[Race Condition]] — определение
- [[Race Detector]] — ловит только DR

|                     | Atomic                       | Mutex                         |
| :------------------ | :--------------------------- | :---------------------------- |
| Что защищает        | **Одну операцию**            | **Блок кода**                 |
| Реализация          | 1 CPU инструкция             | CAS + futex + scheduler       |
| Юзкейс              | Счётчик, флаг, config reload | Несколько полей, любая логика |
| Много горутин сразу | Крутятся в цикле             | Засыпают, ждут пробуждения    |

^atomic-vs-mutex-table

**Под капотом atomic:**

Одна CPU инструкция с блокировкой шины памяти: ^atomic-under-hood

- `LOCK XADD` — для Add ^atomic-lock-xadd
- `LOCK CMPXCHG` — для CompareAndSwap ^atomic-lock-cmpxchg
- `LOCK XCHG` — для Swap ^atomic-lock-xchg

Префикс `LOCK` говорит CPU: "пока я работаю с этой ячейкой памяти, никто другой к ней не лезет". ^atomic-lock-prefix

**Под капотом Mutex:**

Сложнее — комбинация: ^mutex-under-hood

- CAS (atomic) для быстрого пути ^mutex-cas-fast
- Спиннинг (крутимся ~120 циклов) ^mutex-spinning
- Futex (системный вызов) — усыпить/разбудить горутину ^mutex-futex
- Очередь ожидания ^mutex-queue

**Ключевое:**
Mutex = "пока я внутри Lock/Unlock, никто другой туда не зайдёт". ^mutex-guarantee
Atomic = "эта конкретная операция неделима". ^atomic-guarantee

## Связь
- [[sync - atomic]] — API atomic
- [[Структура]] — Mutex под капотом тоже использует atomic
- [[Почему НЕ atomic везде]] — когда atomic не подходит

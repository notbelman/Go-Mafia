- P = логический процессор. Абстракция: права и ресурсы для выполнения Go-кода: локальная очередь G (256, lock-free) + mcache + gFree пул
- GOMAXPROCS = количество P. По умолчанию = NumCPU. Изменение в runtime → **STW**
- mcache на P, не на M — при syscall M блокируется, а P с mcache отдаётся другому M (не простаивает)
- контейнеры: `uber-go/automaxprocs` для корректного определения лимита
- в контейнерах: GOMAXPROCS тянется из хостовой машины, не из лимита контейнера

[[GMP Flashcards - processor]]

**Структура runtime.p:**
```
├── id          — номер процессора (0, 1, 2...)
├── status      — idle, running, syscall, gcstop
├── runq        — локальная очередь G (кольцевой буфер, 256)
├── runnext     — следующая G (приоритетная, одна)
├── mcache      — кэш памяти для аллокаций
├── gFree       — пул свободных G для переиспользования
└── m           — привязанный M (или nil)
```

P — это абстракция: права и ресурсы для выполнения Go-кода. ^p-definition

Состав P: локальная очередь G (256, lock-free) + mcache + gFree пул. ^p-components

**runnext:** только что созданная горутина попадает сюда. Выполнится следующей — лучше локальность кэша. ^p-runnext

**Локальная очередь (runq):** 256 горутин, кольцевой буфер, lock-free для владельца. При переполнении — половина уходит в глобальную очередь. ^p-runq

**mcache:** раньше был на M — проблема: M блокируется на syscall, mcache простаивает. Теперь на P — при syscall P отдаётся с mcache другому M. ^p-mcache-why

**Состояния P:** **idle** (свободен, ждёт M), **running** (привязан к M), **syscall** (M ушёл в syscall, P скоро заберут), **gcstop** (STW). ^p-states

## GOMAXPROCS

```go
runtime.GOMAXPROCS(4)           // установить
n := runtime.GOMAXPROCS(0)      // получить (0 = не менять)
// по умолчанию = runtime.NumCPU()
```

^ebca1e

GOMAXPROCS = количество P = максимальный параллелизм Go-кода. По умолчанию = `runtime.NumCPU()`. ^gomaxprocs-default

Изменение в runtime вызывает **Stop the World**. Лучше не менять на лету. ^gomaxprocs-stw

**Кейс Яндекса с контейнерами:** Docker-контейнер с лимитом 5-6 ядер, но GOMAXPROCS=32 — тянулся из хостовой виртуалки (24-32 ядра). ^gomaxprocs-container-problem

Лишние потоки → контекст-свитчинг ОС → latency/ресурсы хуже на 10-20%. ^gomaxprocs-container-impact

Решение: захардкодить GOMAXPROCS или использовать `uber-go/automaxprocs`. ^gomaxprocs-container-fix

## Связь
- [[GMP обзор]] — роль P в модели
- [[handoff]] — P отсоединяется от M при syscall
- [[work_stealing]] — кража из локальной очереди P
- [[LRQ и GRQ]], [[Внутреннее устройство очередей]]

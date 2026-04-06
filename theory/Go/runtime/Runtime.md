## Runtime в Go

#### [[Concurrency vs Parallelism]]
Concurrency = структура, Parallelism = исполнение — разница

---

## Scheduler (GMP)

#### [[GMP обзор]]
G (goroutine), M (thread), P (processor) — трёхуровневая модель

#### [[Горутины — что это и зачем]]
Горутины vs OS потоки: стек 2KB, M:N scheduling

#### [[G (Goroutine)]]
Структура G: стек, состояние, PC, SP, gopark/goready

#### [[M (Machine)]]
OS-поток, привязка к P, spinning M, thread cache

#### [[P (Processor)]]
Локальная очередь (LRQ), runqhead/runqtail, GOMAXPROCS

#### [[LRQ и GRQ]]
Local run queue (256 горутин) vs Global run queue, приоритет

#### [[Внутреннее устройство очередей]]
Кольцевой буфер, runnext слот

#### [[work_sharing vs work_stealing]]
Различие подходов, Go использует work-stealing

#### [[work_stealing]]
P крадёт горутины из LRQ другого P или GRQ

#### [[handoff]]
M передаёт P другому M при syscall

#### [[Syscalls]]
Entersyscall/exitsyscall, detach P, retake

#### [[netpoller]]
Интеграция с epoll/kqueue, неблокирующий I/O в рантайме

#### [[sysmon]]
Фоновый поток: preemption, retake P, сетевые события

#### [[preemption]]
Кооперативная (Go 1.13-) vs асинхронная сигнальная (Go 1.14+) preemption

#### [[как работают вместе]]
Полный цикл: горутина создана → scheduled → запущена → заблокирована

#### [[Нюансы горутин]]
Стек горутины, segmented vs contiguous stack, goroutine leak

#### [[Паники и горутины]]
Паника не переходит между горутинами, recover в defer

---

## GC (Garbage Collector)

#### [[Обзор GC]]
Tri-color mark-and-sweep, concurrent GC, STW паузы

#### [[Tri-color marking]]
White/grey/black объекты, алгоритм маркировки

#### [[Write barrier]]
Dijkstra/hybrid write barrier, зачем нужен при concurrent mark

#### [[Mark Assist]]
Горутина помогает GC при высоком allocation rate

#### [[GC Pacer]]
Регулятор частоты GC, GOGC, heap goal

#### [[GOGC и GOMEMLIMIT]]
GOGC=100 (по умолчанию), soft memory limit (Go 1.19+)

#### [[Tracing (базовый STW)]]
Stop-the-world фазы: scan stack roots, finalization

#### [[Finalizers]]
runtime.SetFinalizer, когда использовать, подводные камни

#### [[Что такое мусор и когда можно не собирать]]
Reachability, escape analysis, stack allocation = no GC

#### [[Поколения (generational GC)]]
Generational hypothesis, Go не использует поколения (пока)

#### [[Reference counting]]
RC как альтернатива, почему Go не использует

#### [[Lazy allocation и RSS vs VSS]]
Virtual vs resident memory, mmap, lazy commit

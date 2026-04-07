## Scheduler в Go (GMP)

#### [[GMP обзор]]
G (goroutine) + M (OS thread) + P (processor), M:N scheduling

#### [[Горутины — что это и зачем]]
Горутины vs потоки: стек 2KB, дешёвое создание, M:N модель

#### [[G (Goroutine)]]
Структура G: стек, gobuf (SP/PC), статус, schedlink

#### [[M (Machine)]]
OS-поток, g0 (scheduler stack), curg, p, spinning M

#### [[P (Processor)]]
LRQ (256 слотов), runnext, mcache, GOMAXPROCS

#### [[LRQ и GRQ]]
Local run queue vs Global run queue, 1/61 steal from GRQ

#### [[Внутреннее устройство очередей]]
Кольцевой буфер runq[256], head/tail, атомарные операции

#### [[work_sharing vs work_stealing]]
Work-sharing толкает, work-stealing тянет — Go выбрал stealing

#### [[work_stealing]]
P крадёт половину LRQ другого P, порядок: LRQ → GRQ → netpoll → steal

#### [[handoff]]
Syscall: M отдаёт P другому M (или создаёт новый), retake

#### [[Syscalls]]
entersyscall/exitsyscall, blocking vs non-blocking, cgo

#### [[netpoller]]
Интеграция epoll/kqueue в рантайм, gopark на I/O, goready при готовности

#### [[sysmon]]
Фоновый M без P: preempt горутины, retake P от blocked M, netpoll

#### [[preemption]]
Go 1.13-: cooperative (только при function call)
Go 1.14+: async signal-based preemption (SIGURG)

#### [[Нюансы горутин]]
Goroutine leak, runtime.Goexit, GOMAXPROCS, goroutine ID (нет публичного)

#### [[Паники и горутины]]
Паника не пересекает границу горутины, defer/recover локальны

#### [[как работают вместе]]
Полный цикл: go func() → newproc → runqput → schedule → execute

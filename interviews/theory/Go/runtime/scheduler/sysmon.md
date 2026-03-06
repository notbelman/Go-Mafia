sysmon — системный монитор. Отдельный поток ОС, не привязан к P, работает независимо от GOMAXPROCS. ^sysmon-definition

**Задачи sysmon:**

1. **Preemption** — G работает > 10ms → пометить для вытеснения (`g.preempt = true`, SIGURG), вытесняется в GRQ. ^sysmon-preemption
2. **Hand-off** — P в состоянии syscall > 10ms → отдать другому M. ^sysmon-handoff
3. **Netpoll** — проверить готовые сетевые события, разбудить ждущие G(в findRunnable). ^sysmon-netpoll
4. **GC** — запустить GC если не было > 2 минут. ^sysmon-gc
5. **Timers** — проверить сработавшие таймеры. ^sysmon-timers
6. **Fairness** — если две горутины ставят друг друга в runnext (LIFO) и FIFO голодает → вытеснить. ^sysmon-fairness

**Адаптивный сон:** начинает с 20µs. После 50 idle циклов — удваивается, максимум 10ms. Много активности → просыпается чаще. Экономит CPU когда нечего делать. ^sysmon-sleep

[[GMP Flashcards - sysmon]]

**Как посмотреть работу планировщика:**
```bash
GODEBUG=schedtrace=1000 ./myapp          # каждые 1000ms
GODEBUG=schedtrace=1000,scheddetail=1 ./myapp  # подробно
```
^sysmon-debug-cmd

```
SCHED 1000ms: gomaxprocs=4 idleprocs=2 threads=5
  runqueue=0 [3 0 0 1]
              │        └── G в локальных очередях P0-P3
              └── G в глобальной очереди
```
^sysmon-debug-output

## Связь
- [[Preemption]] — как sysmon вытесняет горутины
- [[Handoff]] — как sysmon отвязывает P от заблокированного M
- [[Netpoller]] — sysmon проверяет epoll

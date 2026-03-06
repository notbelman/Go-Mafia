- syscall → M блокируется в ядре → **P отвязывается** от M и отдаётся другому M (из thread pool)
- зачем P отдельная сущность: чтобы при syscall очередь горутин не голодала
- после syscall: G пытается вернуться в свой P; если недоступен → глобальная очередь
- handoff не сразу: short-lived syscalls — M может не откреплятся. sysmon проверяет: syscall > 10ms → handoff
- M остаётся с G потому что ядро привязывает syscall к потоку — нельзя отдать другому M

---

[[GMP Flashcards - handoff]]

Когда M блокируется на syscall — P отдаётся другому M.

**Проблема:** G вызывает syscall (file.Read) → M заблокирован в ядре → P привязан к M → другие G в очереди не выполняются. ^handoff-problem

**Решение — handoff:**
```
1. G1 на M0 вызывает syscall
   P0 ─── M0 [G1 syscall...]
   runq: G2 G3 G4

2. Runtime разрывает связь P0 ↔ M0 (обнуляет указатель)
   P0 (свободен)     M0 [G1 syscall...]

3. P0 привязывается к M1 (из thread pool или новый)
   P0 ─── M1          M0 [G1 syscall...]
   (выполняет G2)

4. Syscall завершён, G1 ищет P:
   ├── свой P0 свободен? → забрать
   ├── любой idle P? → забрать
   └── нет свободного P → G1 в глобальную очередь, M0 в thread pool
```
^handoff-steps

Компилятор знает набор syscalls — перед каждым вставляет код hand-off. Это ещё одна причина, зачем P как отдельная сущность. ^handoff-compiler

**После syscall — поиск P:** сначала пытается вернуться в свой P0, затем ищет любой idle P, иначе G1 идёт в глобальную очередь, M0 паркуется в thread pool. ^handoff-after-syscall

**Когда происходит:** не сразу при каждом syscall. Short-lived syscalls — M может не открепляться от P. ^handoff-not-immediate

sysmon проверяет: P в состоянии syscall > 10ms → handoff. ^handoff-sysmon-threshold

**Thread pool:** потоки переиспользуются. После syscall M паркуется в пул, при следующем hand-off — берётся оттуда. Потоков может быть больше GOMAXPROCS. ^handoff-thread-pool

**Почему M остаётся с G:** syscall выполняется в kernel space, ядро привязывает работу к потоку. Нельзя «отдать» syscall другому потоку. G + M ждут вместе, P отдаётся — там только userspace ресурсы. ^handoff-why-m-stays

**CGO:** компилятор Go не видит C-код, hand-off не произойдёт. CGO запускается в отдельной горутине, но вставки компилятора перед syscall нет. ^handoff-cgo

## Связь
- [[P (Processor)]] — зачем P отдельная сущность
- [[M (Machine)]] — thread pool, переиспользование потоков
- [[Netpoller]] — альтернатива hand-off для сетевых syscalls

- **Syscall** — вызов ядра ОС (файлы, сеть, процессы). В Go два вида: **быстрые** (`RawSyscall`, не отдают P) и **медленные** (`Syscall`, отвязывают M от P чтобы не блокировать планировщик) ^sys-def
- Сетевой I/O — **не syscall** в привычном смысле: Go использует **netpoll** (epoll/kqueue) → горутина паркуется через gopark, M не блокируется ^sys-netpoll
- Файловый I/O — **блокирующий** syscall: M уходит в ядро, P отвязывается и отдаётся другому M ^sys-file-io

---

## Два пути syscall в runtime

### 1. Быстрый: `RawSyscall` / `Syscall6`

Для syscall'ов которые **не блокируются** (getpid, gettimeofday, mmap). ^sys-raw

```
RawSyscall(SYS_GETPID, ...):
  1. Вызвать ядро напрямую
  2. Вернуться — P остаётся привязанным к M
  3. Планировщик даже не знает что был syscall
```

^sys-raw-flow

### 2. Медленный: `Syscall` (через `entersyscall` / `exitsyscall`)

Для syscall'ов которые **могут блокироваться** (read файла, write файла, epoll_wait). ^sys-slow

```
entersyscall():
  1. G.status: _Grunning → _Gsyscall
  2. P отвязывается от M → P свободен, может взять другой M
  3. M уходит в ядро (блокируется)

--- M спит в ядре ---

exitsyscall():
  1. Попробовать вернуть свой P (если свободен)
  2. Если P занят → встать в очередь на любой свободный P
  3. G.status: _Gsyscall → _Grunning
```

^sys-slow-flow

**Ключевое**: пока M спит в ядре, **P не простаивает** — его забирает другой M и продолжает выполнять горутины. ^sys-key

## sysmon — сторожевой поток

**sysmon** — отдельный M без P, крутится в бесконечном цикле. Следит за горутинами застрявшими в syscall. ^sys-sysmon

```
sysmon (каждые 10-20ms):
  - Если G в _Gsyscall дольше 10μs → отобрать P у этого M
  - Передать P другому M (или создать новый M)
  - Также: запуск GC, обработка таймеров, netpoll
```

^sys-sysmon-flow

Без sysmon: одна горутина в блокирующем syscall → весь P стоит. С sysmon: P отбирается и работает. ^sys-sysmon-why

## Сетевой I/O: netpoll (НЕ блокирующий syscall)

```
conn.Read():
  1. Попробовать неблокирующий read → есть данные? вернуть
  2. Нет данных → зарегистрировать fd в epoll/kqueue
  3. gopark (горутина засыпает, M и P свободны!)
  4. epoll сообщил "данные готовы" → goready (горутина просыпается)
```

^sys-netpoll-flow

**M не блокируется** — это главное отличие от файлового I/O. Тысячи горутин могут ждать сеть, занимая всего несколько M. ^sys-netpoll-key

|I/O|Механизм|M блокируется?|P отвязывается?|
|:--|:--|:--|:--|
|**Сеть**|netpoll (epoll/kqueue)|Нет|Нет (gopark)|
|**Файл**|блокирующий syscall|Да|Да (entersyscall)|
|**CGO**|блокирующий syscall|Да|Да|
|^sys-comparison||||

## CGO

Вызов C-кода = блокирующий syscall. `entersyscall` → M уходит в C-код → P отвязывается. Поэтому **CGO дорогой**: каждый вызов = потенциальный новый M + context switch. ^sys-cgo

## Stack trace

```
goroutine 5 [syscall]:            ← M в ядре, блокирующий syscall
goroutine 8 [IO wait]:            ← netpoll, M свободен
goroutine 12 [chan receive]:       ← gopark, не syscall
```

^sys-stacktrace

## Связь

- [[Runtime internals]] — gopark/goready, entersyscall/exitsyscall
- [[Contention]] — блокирующие syscall'ы создают contention за M
- [[worker pool]] — файловый I/O: worker pool ограничивает число M в ядре
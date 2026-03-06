- проблема: 10K соединений → 10K потоков (через hand-off). Решение: epoll + горутина → **waiting**, M свободен
- только **сетевые** syscalls (net.Conn, net.Listener). Файловый I/O → обычный hand-off
-  кто проверяет epoll: P в findRunnable (шаги 5, 7) + sysmon как страховка + M когда им нечего делать ^np-who-checks

---

[[GMP Flashcards - netpoller]]

Netpoller — подсистема для асинхронного сетевого I/O. ^np-definition

**Проблема без netpoller:** I/O-bound сервис, 10K соединений. Каждый read/write — syscall → hand-off → новый поток. 10K потоков, каждый со стеком 2-8MB. Дорого и ломает всю легковесность. ^np-problem

**Решение:** используем мультиплексоры ОС (Linux: epoll, macOS: kqueue, Windows: IOCP). Один поток следит за тысячами сокетов. ^np-solution-os

**Как работает:**
```
1. G вызывает conn.Read()
   
2. Данных нет → G переходит в waiting (не M!)
   fd регистрируется в epoll
   M свободен для других G
   
3. P в findRunnable вызывает netpoll(0) или netpoll(delay)
   (sysmon тоже проверяет, но как страховка)
   
4. Данные пришли → epoll говорит "fd готов" → G становится runnable
   Попадает в очередь P
   
5. P выбирает G, M выполняет чтение
```
^np-flow

Ключевое: **блокируется горутина (waiting), не поток**. M продолжает выполнять другие горутины. ^np-key-insight

**Связь fd и горутины:** файловый дескриптор хранит указатель на G, которая ждёт событие. Всё — структуры данных, легко связать. ^np-fd-g-link

**Что обрабатывает netpoller:** net.Conn (TCP/UDP), net.Listener (Accept), таймеры. ^np-handles

**Что НЕ обрабатывает (блокирует M через hand-off):** файловый I/O (file.Read), CGO вызовы, блокирующие syscall. На epoll крутятся **только сетевые** syscalls. ^np-not-handles

**Эффективность:**
```
Без netpoller: 10K соединений → 10K заблокированных M
С netpoller:   10K соединений → 10K запаркованных G, M свободны
```
^np-efficiency

## Связь
- [[handoff]] — что происходит с несетевыми syscalls
- [[sysmon]] — кто периодически проверяет epoll
- [[Work stealing]] — netpoll как последний шаг поиска работы

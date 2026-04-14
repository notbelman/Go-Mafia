- TCP-соединение = **уникальный tuple** `(src_ip, src_port, dst_ip, dst_port)`. С одного IP на один dst_ip:dst_port — максимум **~28K** соединений (ephemeral ports 32768-60999)
- Реальный лимит чаще **file descriptors**: `ulimit -n` (дефолт 1024), `/proc/sys/fs/file-max` (системный). Каждый сокет = один fd
- **Backlog**: SYN queue (полуоткрытые) + accept queue (установленные, ждут accept()). Переполнение → дропы / SYN flood
- Масштабирование: **несколько src_ip** (×28K каждый), `SO_REUSEADDR`/`SO_REUSEPORT`, увеличить `ulimit`, тюнинг `net.core.somaxconn`

---

## Уникальность соединения

```
Соединение = (src_ip, src_port, dst_ip, dst_port)

Один клиент → один сервер 1.2.3.4:443:
  src_port: 32768..60999 = 28232 порта
  → максимум ~28K одновременных соединений
```

Но: два соединения **могут** иметь одинаковый src_port, если dst_ip или dst_port **разные**:
```
(10.0.0.1, 50000, 1.2.3.4, 443)  ← соединение 1
(10.0.0.1, 50000, 5.6.7.8, 443)  ← соединение 2 (другой dst_ip — ОК)
```

## Что влияет на максимум

| Ресурс | Дефолт | Как проверить | Как увеличить |
|:--|:--|:--|:--|
| Ephemeral ports | 32768-60999 (~28K) | `cat /proc/sys/net/ipv4/ip_local_port_range` | `sysctl net.ipv4.ip_local_port_range="1024 65535"` |
| File descriptors (процесс) | 1024 | `ulimit -n` | `ulimit -n 1000000` или /etc/security/limits.conf |
| File descriptors (система) | 100K-1M | `cat /proc/sys/fs/file-max` | `sysctl fs.file-max=2000000` |
| Accept backlog | 128-4096 | `cat /proc/sys/net/core/somaxconn` | `sysctl net.core.somaxconn=65535` |
| Memory | — | — | Каждый сокет ≈ 3-10 KB (буферы) |

## Backlog: SYN queue + Accept queue

```
Client ──SYN──→ [SYN Queue (полуоткрытые)] ──handshake──→ [Accept Queue] ──accept()──→ приложение
```

**SYN Queue**: соединения после SYN, до завершения handshake. `tcp_max_syn_backlog` (дефолт 128-1024).

**Accept Queue**: handshake завершён, ждут `accept()` от приложения. Размер = `min(backlog в listen(), somaxconn)`.

**Переполнение Accept Queue**: ОС **дропает SYN** или отправляет RST. Клиент видит timeout или connection refused.

```go
// Go: backlog задаётся автоматически через somaxconn
ln, _ := net.Listen("tcp", ":8080")
// внутри: listen(fd, somaxconn)
```

## SYN Flood

Атакующий шлёт SYN с поддельными src_ip → SYN queue забивается → легитимные клиенты не могут подключиться.

**Защита: SYN cookies** — сервер **не хранит состояние** до получения ACK. ISN кодирует информацию о соединении.

```bash
sysctl net.ipv4.tcp_syncookies=1  # включить SYN cookies
```

## SO_REUSEADDR и SO_REUSEPORT

```go
// SO_REUSEADDR: позволяет bind на порт в TIME_WAIT
// Go делает это автоматически для Listen

// SO_REUSEPORT: несколько процессов bind на один порт
// Ядро распределяет входящие соединения между процессами
// В Go: через golang.org/x/sys/unix + syscall.RawConn
```

**SO_REUSEPORT** = несколько accept-горутин/процессов на одном порту → масштабирование accept().

## Масштабирование: миллион соединений (C10M)

```
1 src_ip:          ~28K соединений (ephemeral ports)
4 src_ip:          ~112K
ulimit -n 1000000: до 1M fd
Память:            1M × 10KB = 10GB (буферы)
```

Реальные лимиты: **память** (буферы сокетов) и **CPU** (epoll overhead при миллионах fd).

## Go-специфика

```go
// Проверить текущие лимиты
import "syscall"
var rlimit syscall.Rlimit
syscall.Getrlimit(syscall.RLIMIT_NOFILE, &rlimit)
fmt.Println(rlimit.Cur, rlimit.Max)
```

Go **автоматически** делает сокеты неблокирующими и использует epoll. Миллион горутин с коннектами — ОК, если памяти хватает.

## Связь
- [[TCP — соединение]] — TIME_WAIT влияет на доступные порты
- [[TCP — протокол и гарантии]] — tuple определяет соединение
- [[Сокеты]] — fd, listen, accept, backlog
- [[Syscalls]] — epoll, netpoll в Go runtime

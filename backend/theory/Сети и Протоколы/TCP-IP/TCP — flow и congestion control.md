- **Flow control** — не перегрузить **получателя**. Механизм: **receiver window** (rwnd) — "у меня буфер на N байт, не шли больше". Sliding window
- **Congestion control** — не перегрузить **сеть**. Механизм: **congestion window** (cwnd). Slow start → congestion avoidance → fast retransmit/recovery
- Реальное окно = **min(rwnd, cwnd)**. Flow control ограничивает получателем, congestion — сетью. Оба одновременно
- **Nagle's algorithm**: буферизирует маленькие пакеты (ждёт ACK или MSS). Убивает latency для real-time → отключить: `TCP_NODELAY`
- **TCP Keep-Alive**: проверяет живое ли соединение. Дефолт: первый probe через **2 часа** (слишком долго для прода)

---

## Flow Control (Sliding Window)

```
Отправитель                                  Получатель
    │                                            │
    │ ──── данные (seq=1, 1000 байт) ──────────→ │  буфер: [####______] rwnd=6000
    │ ──── данные (seq=1001, 1000 байт) ────────→│  буфер: [########__] rwnd=2000
    │                                            │
    │ ←─── ACK (ack=2001, window=2000) ──────────│  "у меня осталось 2KB"
    │                                            │
    │ ──── данные (seq=2001, 2000 байт) ────────→│  буфер: [##########] rwnd=0
    │                                            │
    │ ←─── ACK (ack=4001, window=0) ─────────────│  "СТОП! буфер полон"
    │                                            │
    │ ... ждём ...                               │  приложение прочитало данные
    │                                            │
    │ ←─── Window Update (window=8000) ──────────│  "можно снова"
```

**Zero Window** → отправитель шлёт **Window Probe** (1 байт) чтобы узнать когда буфер освободится. Иначе deadlock.

## Congestion Control

**cwnd** (congestion window) — сколько данных отправитель может послать **без подтверждения**. Управляется отправителем на основе потерь.

### Slow Start

```
cwnd = 1 MSS
Каждый ACK: cwnd × 2 (экспоненциальный рост)

RTT 1: отправил 1 сегмент
RTT 2: отправил 2 сегмента
RTT 3: отправил 4 сегмента
RTT 4: отправил 8 сегментов
...пока не достигнем ssthresh или потеря
```

**ssthresh** (slow start threshold) — порог, после которого переходим в congestion avoidance.

### Congestion Avoidance

```
cwnd > ssthresh → линейный рост
Каждый RTT: cwnd += 1 MSS (не удвоение)
```

### При потере пакета

**Timeout** (серьёзная потеря):
```
ssthresh = cwnd / 2
cwnd = 1 MSS
→ обратно в slow start (катастрофа для throughput)
```

**3 duplicate ACK** (fast retransmit):
```
ssthresh = cwnd / 2
cwnd = ssthresh
→ congestion avoidance (быстрое восстановление)
```

### Алгоритмы

| Алгоритм | Когда | Особенность |
|:--|:--|:--|
| **Reno** | классика | slow start → cong. avoidance → fast recovery |
| **CUBIC** | Linux дефолт | агрессивнее, быстрее восстанавливает cwnd |
| **BBR** (Google) | высокоскоростные сети | model-based, не реагирует на потери, меряет bandwidth |

```bash
# Текущий алгоритм
sysctl net.ipv4.tcp_congestion_control
# Сменить
sysctl net.ipv4.tcp_congestion_control=bbr
```

## Nagle's Algorithm

Буферизирует маленькие пакеты: не отправлять пока не получен ACK предыдущего **или** не набрался MSS.

```
Без Nagle: Write(1 байт) → сразу пакет (40 байт header + 1 байт = 41 байт)
С Nagle:   Write(1 байт) → буферизирует → ждёт ACK или MSS → отправляет
```

**Проблема**: интерактивные протоколы (SSH, real-time) — задержка до RTT на каждый маленький пакет.

```go
// Отключить Nagle (включить TCP_NODELAY)
conn.(*net.TCPConn).SetNoDelay(true)  // Go: дефолт true!
```

**Go по дефолту отключает Nagle** (`TCP_NODELAY = true`). Это правильно для большинства серверов.

### Nagle + Delayed ACK = проблема

Delayed ACK: получатель ждёт 40ms перед отправкой ACK (надеется приклеить к ответу). Nagle ждёт ACK. Delayed ACK ждёт данных. → **40ms задержка** на каждый пакет.

## TCP Keep-Alive

Проверяет живое ли соединение (не путать с HTTP Keep-Alive):

```
tcp_keepalive_time  = 7200  (первый probe через 2 часа!)
tcp_keepalive_intvl = 75    (между пробами 75 секунд)
tcp_keepalive_probes = 9    (9 проб → закрыть)
```

**2 часа** — слишком долго для прода. NAT/firewall может убить idle соединение за 5 минут.

```go
conn.(*net.TCPConn).SetKeepAlive(true)
conn.(*net.TCPConn).SetKeepAlivePeriod(30 * time.Second)
```

**Зачем**: обнаружить мёртвый peer (крашнулся, сеть отвалилась) без отправки данных.

## Связь
- [[TCP — протокол и гарантии]] — flow/congestion = часть гарантий
- [[TCP — соединение]] — slow start после каждого нового handshake
- [[TCP — лимиты соединений]] — буферы сокетов = память
- [[WebSocket]] — keep-alive на уровне WS (ping/pong) vs TCP keep-alive

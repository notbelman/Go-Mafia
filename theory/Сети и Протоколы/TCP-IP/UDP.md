- **UDP** — connectionless, **datagram-oriented** (границы сообщений сохраняются), без гарантий доставки/порядка/дубликатов. Header всего **8 байт**
- Где: **DNS** (быстро, один запрос-ответ), **видео/аудио** (потеря кадра лучше задержки), **игры** (latency критичнее), **QUIC/HTTP3**, VPN (WireGuard), IoT, multicast
- "Почему не перевести всё на TCP": **head-of-line blocking** (один потерянный пакет блокирует весь поток), overhead handshake (1.5 RTT), congestion control может быть избыточен
- "Можно ли поверх UDP сделать надёжность": **да** — QUIC (Google), KCP, custom ACK. Проверка целостности файла: hash/checksum (S3 ETag, Content-MD5)
- DNS работает по **UDP:53** (основной) и **TCP:53** (ответ >512 байт, zone transfer)

---

## UDP Header (8 байт)

```
 0      7 8     15 16    23 24    31
├────────┼────────┼────────┼────────┤
│ Source Port     │ Dest Port       │
├─────────────────┼─────────────────┤
│    Length       │   Checksum      │
└─────────────────┴─────────────────┘
```

Минимальный overhead. Сравни: TCP header = 20-60 байт, UDP = 8 байт.

**Checksum** — опциональный в IPv4 (обязательный в IPv6). Проверяет целостность, но **не гарантирует доставку**.

## Datagram-oriented

```
Отправитель:              Получатель:
Send("Hello")   →         Recv() → "Hello"    (точно 5 байт)
Send("World")   →         Recv() → "World"    (точно 5 байт)
```

В отличие от TCP: границы сообщений **сохраняются**. Что отправил — то получил (если дошло).

Но: пакет больше MTU → **фрагментация на IP уровне** → если один фрагмент потерялся → весь датаграм потерян.

## Где UDP лучше TCP

| Use case | Почему UDP | Почему не TCP |
|:--|:--|:--|
| **DNS** | Один запрос-ответ, быстро | Handshake 1.5 RTT = overhead для одного вопроса |
| **Видео/аудио стриминг** | Потерял кадр → пропустил, не задержал весь поток | TCP: потеря → retransmit → **все** последующие кадры ждут (HOL blocking) |
| **Онлайн-игры** | Позиция игрока устарела за 50ms — ретрансмит бесполезен | TCP: retransmit устаревших данных задерживает свежие |
| **QUIC/HTTP3** | Надёжность реализована **поверх** UDP, без HOL blocking | TCP HOL blocking на уровне ОС — не обойти |
| **VPN (WireGuard)** | Минимальный overhead, UDP-in-UDP | TCP-in-TCP = **meltdown** (два congestion control конфликтуют) |
| **Multicast/Broadcast** | TCP не поддерживает multicast | — |

## Почему нельзя "просто использовать TCP везде"

**Head-of-line blocking**: TCP гарантирует порядок. Один потерянный пакет → **все** последующие буферизируются, даже если они для разных потоков/запросов. В HTTP/2 over TCP один потерянный пакет блокирует **все** streams.

**Handshake overhead**: TCP = 1.5 RTT до первых данных. + TLS = ещё 1-2 RTT. Для короткого запроса (DNS) это overhead 3-4×.

**Congestion control**: TCP снижает скорость при потерях. Для стриминга лучше **контролировать самому** (адаптивный битрейт).

## Надёжность поверх UDP

**QUIC** (HTTP/3): надёжность, шифрование, мультиплексирование — всё поверх UDP. Каждый stream **независим** → потеря в одном не блокирует другие.

**Проверка целостности файла** (без TCP-гарантий):
```
S3:  ETag (MD5 hash), Content-MD5 header
App: SHA256 от файла, сравнить с эталоном
```

Можно передать файл по UDP + в конце сверить hash → понять целый или нет.

## UDP в Go

```go
// Сервер
addr, _ := net.ResolveUDPAddr("udp", ":9000")
conn, _ := net.ListenUDP("udp", addr)
buf := make([]byte, 1024)
n, remoteAddr, _ := conn.ReadFromUDP(buf)
conn.WriteToUDP([]byte("pong"), remoteAddr)

// Клиент
conn, _ := net.DialUDP("udp", nil, addr)
conn.Write([]byte("ping"))
```

Нет `Accept()` — нет соединений. Просто `ReadFrom` / `WriteTo`.

## Связь
- [[TCP — протокол и гарантии]] — TCP vs UDP: гарантии vs скорость
- [[DNS — как работает]] — DNS по UDP:53 (основной)
- [[WebSocket]] — WebSocket поверх TCP, но QUIC-based WS обсуждается
- [[Сокеты]] — SOCK_DGRAM = UDP

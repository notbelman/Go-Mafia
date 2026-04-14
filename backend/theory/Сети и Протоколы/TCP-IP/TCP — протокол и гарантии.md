- **TCP** (Transmission Control Protocol) — надёжный, **connection-oriented**, **byte stream** протокол. Гарантирует: доставку, порядок, целостность, flow control, congestion control
- Byte stream: TCP **не знает** про границы сообщений. Отправил 2 раза по 100 байт — получатель может прочитать 1 раз 200 байт. Это **не** message-oriented (в отличие от UDP)
- Header: **20-60 байт** (src/dst port, sequence number, ack number, flags SYN/ACK/FIN/RST, window size, checksum)
- **MTU** = максимальный размер пакета на канальном уровне (Ethernet = **1500 байт**). **MSS** = MTU - 20 (IP) - 20 (TCP) = **1460 байт** полезных данных. Больше MSS → фрагментация
- Каждый байт имеет **sequence number** → получатель собирает в правильном порядке, обнаруживает пропуски, отправляет ACK

| Гарантия | Механизм |
|:--|:--|
| **Доставка** | ACK на каждый сегмент. Нет ACK → retransmit (таймаут или fast retransmit) |
| **Порядок** | Sequence number. Получатель буферизирует out-of-order и собирает |
| **Целостность** | Checksum в header (16-bit, слабый). Плюс CRC на канальном уровне |
| **Нет дубликатов** | По sequence number: "это я уже видел" → отбросить |
| **Flow control** | Receiver window: "у меня буфер на N байт, не шли больше" |
| **Congestion control** | Slow start, congestion avoidance: не перегружать сеть |

## TCP Header (20 байт минимум)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─────────────────────────┼─────────────────────────┤
│      Source Port        │    Destination Port     │
├─────────────────────────┴─────────────────────────┤
│                  Sequence Number                  │
├───────────────────────────────────────────────────┤
│               Acknowledgment Number               │
├──────┼──────┼─┼─┼─┼─┼─┼─┼─────────────────────────┤
│Offset│Reserv│U│A│P│R│S│F│         Window          │
├──────┴──────┴─┴─┴─┴─┴─┴─┼─────────────────────────┤
│       Checksum           │      Urgent Pointer    │
├──────────────────────────┴────────────────────────┤
│                   Options (if any)                │
└───────────────────────────────────────────────────┘
```

**Flags**: SYN (открыть), ACK (подтвердить), FIN (закрыть), RST (сбросить), PSH (протолкнуть в приложение), URG (срочные данные).

## Byte stream vs Message-oriented

```
Отправитель:                    Получатель:
Write("Hello")   ─┐
Write("World")   ─┤──TCP──→    Read() → "HelloWorld"  (одним куском!)
                                или
                                Read() → "Hel"
                                Read() → "loWorld"     (произвольная нарезка)
```

TCP **склеивает и нарезает** как хочет. Для message boundaries нужен протокол уровнем выше (HTTP, gRPC, length-prefix framing).

## MTU и MSS

```
Ethernet MTU = 1500 байт
├── IP header:  20 байт
├── TCP header: 20 байт
└── Payload:    1460 байт ← MSS (Maximum Segment Size)
```

Если приложение пишет 5000 байт → TCP разобьёт на 4 сегмента (1460 + 1460 + 1460 + 620).

**Path MTU Discovery**: ICMP "Packet Too Big" → отправитель уменьшает размер. Если ICMP заблокирован → **black hole** (пакеты пропадают).

## Связь
- [[TCP — соединение]] — handshake, teardown, состояния
- [[TCP — flow и congestion control]] — sliding window, slow start
- [[UDP]] — без гарантий, message-oriented, минимальный overhead
- [[Сокеты]] — SOCK_STREAM = TCP

- **3-way handshake**: SYN → SYN-ACK → ACK (**1.5 RTT** до первых данных). Почему не 2: обе стороны должны подтвердить свой начальный sequence number
- **4-way teardown**: FIN → ACK → FIN → ACK. **TIME_WAIT** (**2×MSL ≈ 60s**): защита от задержавшихся пакетов предыдущего соединения
- **RST** — жёсткий сброс: "соединения не существует" или "не хочу больше общаться". Без TIME_WAIT
- **11 состояний** TCP: LISTEN → SYN_RCVD → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED (и зеркальные)
- **TIME_WAIT** на сервере = проблема: тысячи соединений в TIME_WAIT → ephemeral ports заканчиваются. Решение: `SO_REUSEADDR`

---

## 3-way Handshake (установление)

```
Client                          Server
  │                               │  (LISTEN)
  │──── SYN (seq=100) ──────────→│  (SYN_SENT → SYN_RCVD)
  │                               │
  │←─── SYN-ACK (seq=300,        │
  │      ack=101) ────────────────│
  │                               │
  │──── ACK (ack=301) ──────────→│  (ESTABLISHED ↔ ESTABLISHED)
  │                               │
```

**Почему 3, не 2**: каждая сторона выбирает свой **Initial Sequence Number** (ISN) и должна получить подтверждение. 2 пакета → сервер не знает что клиент получил его ISN.

**ISN рандомный**: защита от подмены пакетов (TCP sequence prediction attack).

**SYN cookie**: защита от SYN flood — сервер не хранит состояние до получения финального ACK.

## 4-way Teardown (закрытие)

```
Client                          Server
  │──── FIN ────────────────────→│  (FIN_WAIT_1 → CLOSE_WAIT)
  │←─── ACK ─────────────────────│  (FIN_WAIT_2)
  │                              │  сервер может ещё отправлять данные
  │←─── FIN ─────────────────────│  (TIME_WAIT ← LAST_ACK)
  │──── ACK ────────────────────→│  (CLOSED)
  │                              │
  │  ~~~ TIME_WAIT (60s) ~~~     │
  │         CLOSED               │
```

**Half-close**: после первого FIN одна сторона закрыла отправку, но ещё **принимает** данные. Go: `conn.(*net.TCPConn).CloseWrite()`.

## TIME_WAIT

Длится **2×MSL** (Maximum Segment Lifetime, обычно 30s → TIME_WAIT = 60s).

**Зачем**: если последний ACK потерялся → сервер ретранслирует FIN → клиент должен быть готов ответить. Без TIME_WAIT: новое соединение на тех же портах получит чужой FIN.

**Проблема на сервере**: активно закрывающая сторона уходит в TIME_WAIT. Сервер закрывает тысячи коротких соединений → тысячи TIME_WAIT → ephemeral ports заканчиваются.

```bash
# Посмотреть TIME_WAIT
ss -s
# или
netstat -an | grep TIME_WAIT | wc -l
```

**Решения:**
- `SO_REUSEADDR` — позволяет bind на порт в TIME_WAIT
- `tcp_tw_reuse = 1` — переиспользовать TIME_WAIT для исходящих
- Клиент закрывает первым (TIME_WAIT уходит на клиента, не на сервер)

## RST (Reset)

Жёсткий сброс без graceful teardown:
- Пакет на закрытый порт → RST
- Приложение крашнулось → ОС отправляет RST
- `SO_LINGER` с timeout=0 → close() отправляет RST вместо FIN
- Firewall/NAT потерял запись о соединении → RST

RST **не** проходит через TIME_WAIT.

## Состояния TCP

```
CLOSED → (client) SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
CLOSED → (server) LISTEN → SYN_RCVD → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

**CLOSE_WAIT** на сервере = приложение **не вызвало** close() после получения FIN. Утечка коннектов. Типичный баг в Go: забыл `defer conn.Close()`.

```bash
# Найти CLOSE_WAIT
ss -tnp | grep CLOSE-WAIT
```

## Связь
- [[TCP — протокол и гарантии]] — что гарантирует TCP
- [[TCP — лимиты соединений]] — TIME_WAIT влияет на доступные порты
- [[TLS — handshake и сертификаты]] — TLS handshake идёт после TCP handshake
- [[Что происходит при curl https]] — полный путь от SYN до ответа

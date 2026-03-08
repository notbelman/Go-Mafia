- **WebSocket** — full-duplex протокол поверх TCP. Начинается как HTTP/1.1 Upgrade → 101 Switching Protocols → **постоянное** двустороннее соединение
- Отличие от HTTP: **persistent** (одно соединение на всё время), **bidirectional** (сервер шлёт без запроса), **low overhead** (frame header 2-14 байт vs HTTP headers на каждый запрос)
- **Ping/Pong** — keep-alive на уровне WS (не путать с TCP keep-alive). Сервер шлёт Ping, клиент обязан ответить Pong
- Когда: **чаты**, real-time уведомления, live dashboards, multiplayer игры. Когда НЕ: REST API, редкие обновления (**SSE** дешевле), request-response
- Go: `gorilla/websocket` (зрелый, maintenance mode) или `nhooyr/websocket` (современный). Горутина на каждый коннект

---

## Handshake (Upgrade)

```
Client → Server:
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Server → Client:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

После 101 — TCP-соединение **переиспользуется** для WebSocket frames. HTTP больше не используется на этом соединении.

**Sec-WebSocket-Key/Accept**: не для безопасности (plain text). Для защиты от случайного upgrade прокси-серверами.

## Frame (формат сообщения)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┼─┼─┼─┼─────────┼─┼─────────────┤
│F│R│R│R│  opcode │M│ Payload len │  Extended payload length ...
│I│S│S│S│  (4)    │A│    (7)      │
│N│V│V│V│         │S│             │
│ │1│2│3│         │K│             │
├─┴─┴─┴─┴─────────┴─┴─────────────┤
│   Masking-key (if MASK=1)        │
├──────────────────────────────────┤
│          Payload Data            │
└──────────────────────────────────┘
```

**Opcodes**: 0x1 = text, 0x2 = binary, 0x8 = close, 0x9 = ping, 0xA = pong.

**Header**: 2-14 байт (vs HTTP: 200-800 байт headers на каждый запрос). Вот почему WS эффективнее для частых мелких сообщений.

## Go: gorilla/websocket

```go
// Сервер
var upgrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool { return true },
}

func handleWS(w http.ResponseWriter, r *http.Request) {
    conn, _ := upgrader.Upgrade(w, r, nil)
    defer conn.Close()

    for {
        msgType, msg, err := conn.ReadMessage()
        if err != nil { break }  // клиент отключился

        conn.WriteMessage(msgType, msg)  // echo
    }
}

http.HandleFunc("/ws", handleWS)
```

```go
// Клиент
conn, _, _ := websocket.DefaultDialer.Dial("ws://localhost:8080/ws", nil)
defer conn.Close()

conn.WriteMessage(websocket.TextMessage, []byte("hello"))
_, msg, _ := conn.ReadMessage()
```

## Ping/Pong

```go
// Сервер: отправлять ping каждые 30 секунд
go func() {
    ticker := time.NewTicker(30 * time.Second)
    for range ticker.C {
        conn.WriteMessage(websocket.PingMessage, nil)
    }
}()

// Клиент: pong автоматически (gorilla/websocket делает сам)
conn.SetPongHandler(func(string) error {
    conn.SetReadDeadline(time.Now().Add(60 * time.Second))
    return nil
})
```

Если Pong не пришёл → соединение мертво → закрыть.

## WebSocket vs SSE vs Long Polling

| | WebSocket | SSE (Server-Sent Events) | Long Polling |
|:--|:--|:--|:--|
| Направление | **bidirectional** | server → client only | bidirectional (костыльно) |
| Протокол | WS (binary) | HTTP (text/event-stream) | HTTP |
| Overhead | 2-14 байт/frame | HTTP headers | HTTP headers каждый poll |
| Reconnect | ручной | **автоматический** (EventSource API) | ручной |
| Через CDN/proxy | Сложно (Upgrade) | Просто (обычный HTTP) | Просто |
| Когда | Чаты, игры, high-freq | Уведомления, ленты, dashboards | Legacy, простота |

**SSE часто лучше** чем WebSocket для server→client: автоматический reconnect, работает через CDN, проще инфраструктура. WebSocket нужен только если клиент **активно отправляет** данные.

## Масштабирование WS

- Каждый WS-коннект = горутина + fd + память (буферы). **100K коннектов ≈ 1-3 GB RAM**
- Load balancer: нужен **sticky sessions** (или отдельный WS endpoint)
- Broadcast: pub/sub через Redis/NATS (один сервер получил событие → раздал своим клиентам)

## Связь
- [[TCP — протокол и гарантии]] — WebSocket работает поверх TCP
- [[TCP — flow и congestion control]] — TCP keep-alive vs WS ping/pong
- [[TLS — handshake и сертификаты]] — wss:// = WebSocket over TLS
- [[Сокеты]] — WS-коннект = TCP сокет после upgrade

- **HTTP** — текстовый (1.1) / бинарный (2/3) протокол запрос-ответ поверх **TCP** (1.1/2) или **UDP/QUIC** (3). Stateless — сервер не помнит предыдущие запросы
- **Анатомия**: start line → headers → **пустая строка `\r\n\r\n`** → body. Тело отделяется от заголовков пустой строкой. Загрузка файлов = `Content-Type: multipart/form-data`
- **HTTPS**: TLS шифрует **всё** (path, headers, body). В открытом виде только **IP** (сетевой уровень) и **SNI** (имя домена в TLS ClientHello, ECH решает)
- **Virtual hosting**: несколько сайтов на одном IP:port → сервер различает по **Host header** (HTTP/1.1) или **:authority** (HTTP/2) + **SNI** в TLS
- **HTTP/1.1** → **HTTP/2** (бинарный, мультиплексинг, HPACK) → **HTTP/3** (QUIC/UDP, нет TCP HOL blocking, 0-RTT)

---

## Анатомия запроса

```
POST /api/users HTTP/1.1          ← start line (метод, path, версия)
Host: example.com                  ← headers
Content-Type: application/json
Authorization: Bearer eyJhbG...
Content-Length: 42
                                   ← пустая строка (\r\n\r\n) — разделитель
{"name": "Ivan", "age": 30}       ← body
```

## Анатомия ответа

```
HTTP/1.1 201 Created              ← status line (версия, код, reason)
Content-Type: application/json
Location: /api/users/123
Set-Cookie: session=abc; HttpOnly
                                   ← пустая строка
{"id": 123, "name": "Ivan"}       ← body
```

## Загрузка файлов (multipart)

```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="photo"; filename="cat.jpg"
Content-Type: image/jpeg

<бинарные данные фото>
------FormBoundary
Content-Disposition: form-data; name="description"

My cat
------FormBoundary--
```

Фотка передаётся **в body** как часть multipart. Каждая часть имеет свой Content-Type.

## HTTPS — что шифруется

```
В открытом виде:
  - IP-адреса (src/dst) — сетевой уровень, ниже TLS
  - SNI (имя домена) — TLS ClientHello (ECH в будущем зашифрует)

Зашифровано (всё остальное):
  - Path (/api/users/123)
  - Headers (Authorization, Cookie, Host)
  - Body (JSON, файлы)
  - Query string (?password=secret)
```

**Зачем шифровать**: без TLS любой на пути (WiFi, провайдер, прокси) видит пароли, токены, данные.

## Virtual Hosting

Несколько сайтов на одном IP:443:

```
HTTP/1.1: сервер смотрит Host header
  Host: site-a.com → отдать site-a
  Host: site-b.com → отдать site-b

TLS: сервер смотрит SNI в ClientHello (до расшифровки!)
  SNI: site-a.com → выбрать сертификат site-a
```

Без Host header (HTTP/1.0) и без SNI → сервер не знает какой сайт → отдаёт дефолтный.

## HTTP/1.1 (1997)

- **Keep-Alive**: одно TCP для нескольких запросов (дефолт)
- **Pipelining**: посылать запросы не дожидаясь ответа (на практике **не работает** — прокси ломают)
- **HOL Blocking**: один медленный ответ блокирует **всю** очередь
- Workaround: domain sharding (cdn1.example.com, cdn2.example.com) — больше TCP-соединений

## HTTP/2 (2015)

- **Бинарный** протокол (не текстовый) → эффективнее парсинг
- **Multiplexing**: несколько streams в **одном TCP** параллельно
- **HPACK**: сжатие headers (повторяющиеся headers не передаются)
- **Server Push**: сервер отправляет style.css **до** запроса клиента
- Требует **HTTPS** в браузерах (технически не обязательно)
- Проблема: **TCP HOL Blocking** остаётся — потеря одного пакета блокирует **все** streams

```
HTTP/1.1: |--req1--|--resp1--|--req2--|--resp2--|  (последовательно)
HTTP/2:   |--req1--|--req2--|                       (параллельно в одном TCP)
          |--resp2--|--resp1--|
```

**Что в бинарном формате**: frames (DATA, HEADERS, SETTINGS, PUSH_PROMISE). Каждый frame имеет stream ID → мультиплексинг.

## HTTP/3 (2022)

- **QUIC** вместо TCP (поверх **UDP**)
- **Нет TCP HOL Blocking**: потеря пакета блокирует только один stream, не все
- **0-RTT resume**: повторное подключение без handshake (данные в первом пакете)
- **Connection Migration**: смена WiFi → LTE без разрыва (идентификация по connection ID, не по IP:port)
- Встроенный TLS 1.3 (нельзя без шифрования)
- Минус: UDP может блокироваться файрволами, больше CPU

## Какая самая распространённая

HTTP/1.1 всё ещё **самая распространённая** по количеству серверов. HTTP/2 — по трафику (крупные сайты). HTTP/3 растёт (Google, Cloudflare, Facebook).

## HTTP поверх чего

| Версия | Транспорт |
|:--|:--|
| HTTP/1.1 | TCP |
| HTTP/2 | TCP |
| HTTP/3 | **UDP** (QUIC) |

**Может ли HTTP работать по UDP?** HTTP/1.1 и 2 — нет, только TCP. HTTP/3 — да, QUIC поверх UDP.

## Связь
- [[TCP — протокол и гарантии]] — HTTP/1.1 и 2 поверх TCP
- [[UDP]] — HTTP/3 поверх QUIC (UDP)
- [[TLS — handshake и сертификаты]] — HTTPS = HTTP + TLS, SNI
- [[HTTP методы]] — GET/POST/PUT/PATCH/DELETE
- [[Что происходит при curl https]] — полный путь запроса

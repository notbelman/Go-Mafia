## HTTP/1.0 (1996)
- Новое TCP соединение для каждого запроса
- Headers, статус-коды, POST/HEAD
- Медленный (каждый запрос = новый TCP handshake)

## HTTP/1.1 (1997) - основная версия сегодня
- Keep-Alive: одно TCP для нескольких запросов
- Pipelining (не работает на практике)
- PUT, PATCH, DELETE, OPTIONS
- Cache-Control, ETag, chunked encoding

Проблема: Head-of-Line Blocking (один медленный запрос блокирует очередь)
Workaround: domain sharding (cdn1.example.com, cdn2.example.com)

## HTTP/2 (2015)
- Бинарный протокол (не текстовый)
- Multiplexing: несколько запросов параллельно в одном TCP
- Header Compression (HPACK): headers сжимаются
- Server Push: сервер шлёт /style.css до запроса клиента

Проблема: Head-of-Line Blocking на уровне TCP (потеря пакета блокирует все streams)
Требует HTTPS (браузеры)

## HTTP/3 (2022)
- QUIC вместо TCP (UDP-based)
- Нет TCP Head-of-Line Blocking: потеря пакета не блокирует другие streams
- 0-RTT: повторное подключение без handshake (мгновенно)
- Connection Migration: смена IP/сети без разрыва соединения

Проблема: UDP может блокироваться файрволами, больше CPU

## Главные отличия
HTTP/1.1: один запрос блокирует другие (HOL blocking)
HTTP/2: multiplexing в одном TCP (но TCP HOL blocking остаётся)
HTTP/3: QUIC (UDP) - нет HOL blocking вообще
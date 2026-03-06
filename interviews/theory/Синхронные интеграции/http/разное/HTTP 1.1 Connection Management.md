Connection: keep-alive  - переиспользование TCP (default в HTTP/1.1)
Connection: close       - закрыть после ответа

## Проблемы HTTP/1.1
Head-of-line blocking: 1 медленный запрос блокирует очередь
Workaround: domain sharding (поддомены) или HTTP/2

## HTTP/2
Multiplexing: несколько запросов в 1 TCP
Server Push: сервер шлёт /style.css до запроса клиента
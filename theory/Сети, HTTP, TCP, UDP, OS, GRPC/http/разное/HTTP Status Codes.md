1xx - Informational (100 Continue, 101 Switching Protocols)
2xx - Success (200 OK, 201 Created, 204 No Content)
3xx - Redirection (301 Moved Permanently, 302 Found, 304 Not Modified)
4xx - Client Error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict)
5xx - Server Error (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable)

## Частые ошибки
- 401 vs 403: 401 - "не залогинен", 403 - "залогинен, но нет прав"
- 404 vs 410: 404 - "не найдено сейчас", 410 - "удалено навсегда"
- 502 vs 503: 502 - "upstream сдох", 503 - "сервис перегружен/в maintenance"
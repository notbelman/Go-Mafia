## Request Headers
Host: example.com (обязательный в HTTP/1.1)
User-Agent: Mozilla/5.0... (браузер/клиент)
Accept: application/json (ожидаемый формат)
Authorization: Bearer <token> (аутентификация)
Content-Type: application/json (тип тела запроса)
Cookie: session_id=abc123 (cookies)

## Response Headers
Content-Type: application/json (тип ответа)
Content-Length: 1234 (размер тела в байтах)
Cache-Control: max-age=3600 (кеширование)
Set-Cookie: session_id=abc; HttpOnly; Secure
Location: /users/123 (редирект или созданный ресурс)
ETag: "v1" (версия ресурса для валидации)

## Conditional Requests

If-None-Match: "v1" (GET с ETag - вернет 304 Not Modified если не изменилось)
If-Match: "v1" (PUT/PATCH - вернет 412 Precondition Failed если ETag не совпадает)
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
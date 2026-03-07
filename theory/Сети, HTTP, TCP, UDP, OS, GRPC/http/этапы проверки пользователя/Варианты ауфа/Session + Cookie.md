Stateful: сервер хранит сессии в БД/Redis, клиент хранит session_id в cookie.

## Flow
1. POST /login -> сервер создает session_id="abc123"
2. Сервер сохраняет в Redis: session:abc123 = {user_id: 456, role: "admin"}
3. Сервер отправляет: Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict
4. Браузер автоматически шлет Cookie: session_id=abc123 в каждом запросе
5. Сервер достает из Redis session:abc123 -> user_id=456

## Cookie атрибуты
HttpOnly: защита от XSS (JS не может прочитать cookie)
Secure: только HTTPS
SameSite=Strict: защита от CSRF (cookie не отправится с другого сайта)
Max-Age=3600: время жизни в секундах

## Плюсы
- Легко отозвать: DELETE session:abc123 из Redis
- Меньше размер данных в запросе
- Браузер автоматически управляет cookie

## Минусы
- Stateful: нужен Redis/БД для хранения сессий
- Сложнее масштабировать (sticky sessions или shared Redis)
- CSRF уязвимость (workaround: CSRF token)
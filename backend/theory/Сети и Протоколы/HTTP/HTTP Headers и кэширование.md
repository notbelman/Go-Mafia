- **Request**: `Host` (обязателен в 1.1), `Authorization` (Bearer token), `Content-Type` (тип body), `Accept` (ожидаемый формат), `Cookie`
- **Response**: `Content-Type`, `Set-Cookie` (HttpOnly, Secure, SameSite), `Cache-Control`, `ETag`, `Location` (redirect/created)
- **Кэширование**: `Cache-Control: max-age=3600` (на сколько кэшировать) → `ETag` + `If-None-Match` → **304 Not Modified** (тело не слать). `no-cache` = проверять сервер, `no-store` = не кэшировать
- **CORS**: браузер блокирует fetch с другого домена. **Preflight** (OPTIONS) перед POST/PUT/DELETE: `Access-Control-Allow-Origin`, `Allow-Methods`, `Allow-Headers`
- **Куки**: `Set-Cookie` в ответе → браузер **автоматически** шлёт `Cookie` в каждом запросе на этот домен. HttpOnly = JS не видит, Secure = только HTTPS, SameSite = защита от CSRF

---

## Ключевые Request Headers

| Header | Зачем | Пример |
|:--|:--|:--|
| `Host` | Обязательный (HTTP/1.1). Virtual hosting | `Host: example.com` |
| `Authorization` | Аутентификация | `Bearer eyJhbG...` |
| `Content-Type` | Тип body запроса | `application/json`, `multipart/form-data` |
| `Accept` | Какой формат ответа хочу | `application/json` |
| `Cookie` | Куки (автоматически браузером) | `session_id=abc123` |
| `If-None-Match` | Условный запрос (ETag) | `"v1"` → 304 если не изменился |
| `If-Modified-Since` | Условный запрос (дата) | `Wed, 21 Oct 2025 07:28:00 GMT` |
| `User-Agent` | Клиент | `Mozilla/5.0...` |

## Ключевые Response Headers

| Header | Зачем | Пример |
|:--|:--|:--|
| `Content-Type` | Тип body ответа | `application/json; charset=utf-8` |
| `Content-Length` | Размер body в байтах | `1234` |
| `Set-Cookie` | Установить куку | `session=abc; HttpOnly; Secure` |
| `Cache-Control` | Правила кэширования | `public, max-age=3600` |
| `ETag` | Версия ресурса | `"v1"` |
| `Location` | URL созданного ресурса / редирект | `/users/123` |
| `Retry-After` | Когда повторить (429, 503) | `60` (секунд) |

## Кэширование

```
Первый запрос:
GET /api/users/123
→ 200 OK
→ Cache-Control: max-age=3600
→ ETag: "abc123"

Повторный запрос (через 30 мин — кэш ещё валиден):
→ Браузер берёт из кэша, запрос НЕ уходит на сервер

Повторный запрос (через 2 часа — кэш протух):
GET /api/users/123
If-None-Match: "abc123"
→ 304 Not Modified (ресурс не изменился, тело не шлём — экономия трафика)
или
→ 200 OK + новый ETag (ресурс изменился)
```

### Cache-Control директивы

| Директива | Что значит |
|:--|:--|
| `public, max-age=3600` | CDN + браузер кэшируют на 1 час |
| `private, max-age=3600` | Только браузер (не CDN). Персональные данные |
| `no-cache` | Кэшировать можно, но **проверять сервер** каждый раз (ETag) |
| `no-store` | **Не кэшировать** вообще (пароли, банковские данные) |

### Content Negotiation

```
Accept: application/json           → сервер отдаёт JSON
Accept: application/xml            → сервер отдаёт XML
Accept-Language: ru-RU, en;q=0.9   → русский приоритетнее
Accept-Encoding: gzip, br          → сервер сжимает ответ
```

## Куки — как работают

```
1. Сервер: Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=3600
2. Браузер сохраняет куку
3. Каждый запрос на тот же домен: Cookie: session_id=abc123 (автоматически!)
```

| Атрибут | Защита |
|:--|:--|
| `HttpOnly` | JS не может прочитать (`document.cookie`) → защита от **XSS** |
| `Secure` | Только по HTTPS |
| `SameSite=Strict` | Не отправлять с другого сайта → защита от **CSRF** |
| `Max-Age=3600` | Время жизни (секунды). Без него — session cookie (до закрытия браузера) |

## CORS

Браузер блокирует `fetch("https://api.other.com")` с `https://frontend.com` без разрешения сервера.

**Simple requests** (GET/HEAD/POST + простые headers) → без preflight.

**Preflight** (POST с `Content-Type: application/json`, PUT, DELETE, кастомные headers):
```
Браузер → OPTIONS /api/users
Origin: https://frontend.com

Сервер → 200 OK
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400   ← кэш preflight на 24ч
```

**CORS — только браузер**. Curl, Go http.Client — не проверяют CORS.

## Связь
- [[HTTP протокол и версии]] — headers = часть анатомии запроса/ответа
- [[HTTP Status Codes]] — 304 Not Modified при валидации кэша
- [[Аутентификация и авторизация]] — Authorization header, Cookie

- **1xx** — информационные (100 Continue, 101 Switching Protocols для WebSocket). **2xx** — успех (200 OK, 201 Created, 204 No Content)
- **3xx** — редиректы: **301** (навсегда, кэшируется), **302** (временно), **304 Not Modified** (кэш валиден, тело не слать)
- **4xx** — ошибка клиента: **401** = "не залогинен" (Unauthorized), **403** = "залогинен, но нет прав" (Forbidden), **404** = не найдено, **409** Conflict, **429** Too Many Requests (rate limit)
- **5xx** — ошибка сервера: **500** Internal, **502** Bad Gateway (upstream умер), **503** Service Unavailable (overload/maintenance), **504** Gateway Timeout
- Структура: первая цифра = **класс** ошибки. Помнить все не нужно, нужно понимать **классы**

---

## 1xx — Informational

| Код | Когда |
|:--|:--|
| **100 Continue** | Клиент отправил headers с `Expect: 100-continue`, сервер говорит "шли body". Полезно для больших upload'ов — проверить авторизацию до отправки 2GB файла |
| **101 Switching Protocols** | WebSocket upgrade: HTTP → WS |

## 2xx — Success

| Код | Когда |
|:--|:--|
| **200 OK** | GET/PUT/PATCH — всё ОК, вот ответ |
| **201 Created** | POST — ресурс создан. + `Location: /users/123` |
| **204 No Content** | DELETE — удалено, тела нет |
| **202 Accepted** | Запрос принят, но ещё не обработан (async операция) |

## 3xx — Redirection

| Код | Когда | Кэш |
|:--|:--|:--|
| **301 Moved Permanently** | URL изменился навсегда | Кэшируется |
| **302 Found** | Временный редирект | Не кэшируется |
| **304 Not Modified** | Клиент прислал `If-None-Match` / `If-Modified-Since` → ресурс не изменился → **тело не слать** | — |
| **307 Temporary Redirect** | Как 302, но **сохраняет метод** (POST → POST) |
| **308 Permanent Redirect** | Как 301, но **сохраняет метод** |

**301/302 ловушка**: браузер может **сменить метод** на GET при редиректе POST. 307/308 гарантируют сохранение метода.

## 4xx — Client Error

| Код | Когда |
|:--|:--|
| **400 Bad Request** | Невалидный JSON, плохие параметры |
| **401 Unauthorized** | **Не аутентифицирован** ("кто ты?"). Нужен Authorization header |
| **403 Forbidden** | **Аутентифицирован, но нет прав** ("ты не админ") |
| **404 Not Found** | Ресурс не найден |
| **405 Method Not Allowed** | POST на endpoint который только GET |
| **409 Conflict** | Конфликт (дубликат email, оптимистичная блокировка) |
| **410 Gone** | Удалён **навсегда** (в отличие от 404 — "не найден сейчас") |
| **412 Precondition Failed** | If-Match / If-Unmodified-Since не совпал (ETag) |
| **422 Unprocessable Entity** | Синтаксис ОК, но семантически невалидно |
| **429 Too Many Requests** | Rate limit. + `Retry-After` header |

**401 vs 403**: "ты не показал паспорт" vs "паспорт показал, но ты в чёрном списке".

## 5xx — Server Error

| Код | Когда |
|:--|:--|
| **500 Internal Server Error** | Необработанная ошибка на сервере (panic, nil pointer) |
| **502 Bad Gateway** | Прокси/LB не получил ответ от upstream (upstream крашнулся) |
| **503 Service Unavailable** | Сервис перегружен или в maintenance. + `Retry-After` |
| **504 Gateway Timeout** | Прокси/LB — upstream не ответил вовремя |

**502 vs 504**: 502 = upstream **ответил плохо** (connection refused, bad response). 504 = upstream **не ответил вовремя** (timeout).

## Связь
- [[HTTP методы]] — какой код для какого метода (201 для POST, 204 для DELETE)
- [[Идемпотентность HTTP]] — 409 Conflict, 412 Precondition Failed
- [[HTTP Headers и кэширование]] — 304 Not Modified, ETag, Cache-Control

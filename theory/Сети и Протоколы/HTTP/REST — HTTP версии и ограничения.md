- REST **не привязан к HTTP**. Но HTTP идеально ложится: методы=CRUD, URI=ресурсы, status codes=результат, headers=метаданные. Поэтому REST = HTTP на практике
- REST **работает** на HTTP/1.1, HTTP/2 и HTTP/3 **одинаково**. HTTP/2 даёт мультиплексинг (быстрее параллельные запросы), но REST этого **не замечает** — upgrade прозрачен
- Почему REST **не на HTTP/2 by default**: REST = запрос-ответ (request-response), не использует streaming/push — это территория gRPC и WebSocket. HTTP/2 фичи (streams, server push) REST'у **не нужны**
- **Ограничения REST**: over-fetching, under-fetching, N+1 запросов, нет real-time (подписок), нет строгой типизации. Когда это критично — **GraphQL, gRPC, WebSocket**

---

## REST на разных версиях HTTP

| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|:--|:--|:--|:--|
| REST работает? | ✅ | ✅ | ✅ |
| Что меняется для REST | — | Быстрее (мультиплексинг) | Ещё быстрее (нет TCP HOL) |
| Streaming | ❌ | Есть, но REST не использует | Есть, но REST не использует |
| Server Push | ❌ | Есть, но REST не использует | Есть |

**HTTP/2 для REST**: клиент шлёт 10 параллельных GET → HTTP/1.1 открывает 6 TCP-соединений (browser limit) → HTTP/2 мультиплексирует в одном TCP. REST-код **не меняется**, транспорт быстрее.

```
HTTP/1.1:  [GET /users]──[resp]──[GET /orders]──[resp]──  (последовательно)
HTTP/2:    [GET /users]──[GET /orders]──                    (параллельно)
           [resp users]──[resp orders]──
```

## Почему REST "живёт" на HTTP/1.1

REST = **request-response**. Одна пара: запрос → ответ. Это всё.

HTTP/2 добавляет:
- **Streaming** — REST не использует (один запрос = один ответ)
- **Server Push** — REST не использует (клиент сам решает что запросить)
- **Бинарный формат** — прозрачно, REST не замечает

gRPC **использует** HTTP/2 streaming (bidirectional, server-side, client-side). REST — нет. Поэтому gRPC **требует** HTTP/2, а REST **работает** на чём угодно.

**Можно ли REST на HTTP/2?** Да, и нужно. Nginx/CDN автоматически апгрейдят. Приложение не меняется. Клиент получает мультиплексинг бесплатно.

## Ограничения REST

### Over-fetching

```
GET /users/123 → {"id": 123, "name": "Ivan", "email": "...", "avatar": "...",
                   "bio": "...", "settings": {...}, "created_at": "..."}

Нужно было только name. Получил всё.
```

Решение: `?fields=id,name` (partial response) — но каждый API реализует по-своему.

### Under-fetching

```
Страница профиля: нужны user + orders + reviews

GET /users/123           ← запрос 1
GET /users/123/orders    ← запрос 2
GET /users/123/reviews   ← запрос 3

3 запроса вместо одного. 3 RTT.
```

GraphQL решает одним запросом: `{ user(id: 123) { name, orders { id }, reviews { id } } }`.

### N+1 запросов

```
GET /orders → [{id: 1, user_id: 10}, {id: 2, user_id: 20}, ...]

Для каждого заказа:
GET /users/10
GET /users/20
...
100 заказов = 101 запрос
```

Решения в REST: `?expand=user` (embed), `?include=users` (JSON:API sideloading).

### Нет real-time

REST = pull (клиент спрашивает). Для push нужны:
- **Polling**: `GET /notifications` каждые 5 секунд — неэффективно
- **Long polling**: сервер держит соединение пока нет данных — костыль
- **SSE**: server-sent events — server → client stream
- **WebSocket**: full-duplex — правильное решение для real-time

### Нет строгой типизации

REST + JSON: `{"age": "25"}` — строка или число? Клиент узнает в runtime.

gRPC (.proto) / GraphQL (schema): типы проверяются **до** запуска. Code generation.

## Когда REST не подходит

| Проблема | Альтернатива |
|:--|:--|
| Over/under-fetching, сложный UI | **GraphQL** |
| Internal high-load, streaming | **gRPC** |
| Real-time, bidirectional | **WebSocket** |
| Server → client events | **SSE** |
| Простой internal RPC | **JSON-RPC** |

**REST = default choice**. Уходи от REST только когда **конкретная** проблема REST'а мешает.

## Связь
- [[REST — архитектурный стиль]] — 6 constraints, Richardson levels
- [[REST — дизайн API]] — naming, versioning, pagination
- [[HTTP протокол и версии]] — HTTP/1.1 vs 2 vs 3 подробнее
- [[WebSocket]] — real-time альтернатива REST

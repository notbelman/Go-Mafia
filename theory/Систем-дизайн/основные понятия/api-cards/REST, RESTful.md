**Суть:** Архитектурный стиль API. Запрос = **глагол** (HTTP метод) + **существительное** (URI ресурса).

**Структура запроса:**
- Метод: GET, POST, PUT, DELETE
- URI: /users/5, /orders/123
- Headers: метаинформация (Content-Type, Authorization)
- Body: данные в JSON (для POST, PUT, PATCH)

**Коды ответа:**
- 2xx — успех (200 OK, 201 Created, 204 No Content)
- 3xx — перенаправление
- 4xx — ошибка клиента (400 Bad Request, 401, 403, 404)
- 5xx — ошибка сервера (500, 502, 503)

**REST vs RESTful:**
- REST — концепция, набор принципов
- RESTful — API, которое полностью соблюдает все принципы REST
- На практике мало кто делает строго RESTful — обычно "REST-like"

**Халивары:**
- GET vs POST для расчётов (GET /rides/estimate или POST /rides/estimate)
- DELETE vs PATCH для отмены (DELETE /orders/123 или PATCH /orders/123 {canceled: true})
- Правильного ответа нет — зависит от команды и контекста

**Когда использовать:** по умолчанию для внешних API (клиент → сервер). Самый распространённый стиль.

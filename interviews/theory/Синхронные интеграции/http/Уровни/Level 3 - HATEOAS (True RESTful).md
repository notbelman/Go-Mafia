Сервер возвращает готовые URL для всех доступных действий.

GET /users/123
{
  "id": 123,
  "name": "Ivan",
  "status": "active",
  "_links": {
    "self": "/users/123",
    "orders": "/users/123/orders",
    "deactivate": "/users/123/deactivate"
  }
}

Клиент НЕ хардкодит URL, следует ссылкам (как браузер по HTML).

Если статус "banned":
{
  "id": 123,
  "status": "banned",
  "_links": {
    "self": "/users/123",
    "activate": "/users/123/activate"
  }
}
Нет ссылок edit/delete - нельзя редактировать забаненного.

## Почему Level 3 не используется
- Сложность: клиент должен парсить _links, а не хардкодить URL
- Оверхед: каждый response раздувается ссылками
- Frontend всё равно хардкодит URL на практике
- GraphQL удобнее если нужна гибкость

## Итого
REST-like (Level 2) - стандарт индустрии
RESTful (Level 3) - теоретический идеал, редко встречается
На собесе "RESTful API" обычно означает Level 2
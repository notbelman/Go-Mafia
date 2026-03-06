Создаёт НОВЫЙ ресурс. Сервер генерит ID.

```
POST /users
{"name": "Ivan", "email": "ivan@example.com"}

-> 201 Created
-> Location: /users/123
```

Не идемпотентен: каждый запрос создаст нового юзера.

## Другие use-cases POST
- Поиск с телом запроса: POST /search {"query": "..."}
- RPC-style операции: POST /calculate {"a": 5, "b": 10}
- Действия: POST /orders/123/cancel

## Как сделать идемпотентным
Idempotency-Key header или уникальный бизнес-ключ (email).
Изменяет ТОЛЬКО указанные поля.

```
PATCH /users/123
{"email": "new@example.com"}
```

-> Изменит только email, остальные поля не тронет

## Идемпотентность зависит от реализации

Идемпотентно (SET):
PATCH /users/123 {"email": "new@example.com"}
-> N запросов = email всегда "new@example.com"

НЕ идемпотентно (INCREMENT):
PATCH /users/123 {"balance": "+100"}
-> N запросов = balance += 100 каждый раз

## JSON Patch (RFC 6902)
PATCH /users/123
[
  {"op": "replace", "path": "/email", "value": "new@example.com"},
  {"op": "add", "path": "/age", "value": 30}
]

Операции: add, remove, replace, move, copy, test
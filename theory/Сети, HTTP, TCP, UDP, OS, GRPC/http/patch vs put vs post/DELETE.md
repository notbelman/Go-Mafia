Удаляет ресурс.

DELETE /users/123

-> 204 No Content (удалён, тела нет)
-> 200 OK (удалён, вернули инфо о ресурсе)
-> 404 Not Found (повторный DELETE - ресурса уже нет)

Идемпотентен: N DELETE = 1 DELETE (эффект тот же - ресурса нет)

## Soft delete vs Hard delete
Hard delete: DELETE FROM users WHERE id=123
Soft delete: UPDATE users SET deleted_at=NOW() WHERE id=123
-> Для soft delete иногда используют PATCH вместо DELETE

## Можно ли вернуть тело?
Да, но не обязательно. Примеры:
200 OK + {"id": 123, "name": "Ivan"} (возврат удалённого)
204 No Content (стандартный вариант)
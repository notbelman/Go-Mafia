- **GET** (read, safe, idempotent, cacheable), **POST** (create, НЕ idempotent), **PUT** (полная замена, idempotent), **PATCH** (частичное изменение, НЕ idempotent по спеке), **DELETE** (удаление, idempotent)
- **Safe** = не меняет состояние (GET, HEAD, OPTIONS). **Idempotent** = N запросов = 1 запрос (GET, PUT, DELETE). **Cacheable** = ответ можно кэшировать (GET, HEAD)
- REST: один URL `/users/123` + разные методы вместо `/getUser`, `/createUser`, `/deleteUser`
- **curl** без флагов = **GET**. С `-d` = **POST**. С `-X PUT` = PUT

---

## Сводная таблица

| Метод | CRUD | Safe | Idempotent | Cacheable | Body | Когда |
|:--|:--|:--|:--|:--|:--|:--|
| **GET** | Read | ✅ | ✅ | ✅ | Нет | Получить ресурс |
| **POST** | Create | ❌ | ❌ | ❌* | Да | Создать / RPC / поиск |
| **PUT** | Replace | ❌ | ✅ | ❌ | Да | Заменить ресурс целиком |
| **PATCH** | Update | ❌ | ❌** | ❌ | Да | Частичное изменение |
| **DELETE** | Delete | ❌ | ✅ | ❌ | Опц. | Удалить ресурс |
| **HEAD** | Read | ✅ | ✅ | ✅ | Нет | Метаданные без тела |
| **OPTIONS** | — | ✅ | ✅ | ❌ | Нет | CORS preflight, доступные методы |

*POST cacheable по спеке если Cache-Control, но никто не делает.
**PATCH: `{email: "x"}` = идемпотентно, `{balance: "+100"}` = нет.

## GET

```
GET /users/123 → 200 OK {"id": 123, "name": "Ivan"}
```

Параметры в **URL** (query string), не в body. Не должен менять состояние. Браузер может prefetch GET-запросы.

## POST — создание

```
POST /users
{"name": "Ivan", "email": "ivan@example.com"}
→ 201 Created + Location: /users/123
```

**Не идемпотентен**: каждый запрос создаёт нового юзера. Retry = дубликат. Решение: Idempotency-Key.

Другие use cases: поиск с телом (`POST /search`), RPC-операции (`POST /orders/123/cancel`).

## PUT — полная замена

```
PUT /users/123
{"name": "Ivan", "email": "new@example.com", "role": "admin"}
→ 200 OK
```

Заменяет **целиком**. Не передал поле → **удалится/обнулится**. Upsert: если нет → создаст (201), есть → заменит (200).

## PATCH — частичное изменение

```
PATCH /users/123
{"email": "new@example.com"}
→ 200 OK (изменил ТОЛЬКО email)
```

JSON Patch (RFC 6902): `[{"op": "replace", "path": "/email", "value": "new@mail.com"}]`.

## DELETE

```
DELETE /users/123
→ 204 No Content
```

Идемпотентен: первый раз удалит (204), повторный → 404 (ресурса нет), но **состояние то же** — ресурса нет.

Soft delete: `UPDATE SET deleted_at = NOW()` — иногда используют PATCH вместо DELETE.

## HEAD

Как GET, но **без тела** ответа. Только headers. Use case: проверить Content-Length перед скачиванием большого файла.

## OPTIONS

```
OPTIONS /api/users
→ Allow: GET, POST, OPTIONS
→ Access-Control-Allow-Methods: GET, POST  (CORS)
```

## Связь
- [[HTTP протокол и версии]] — методы = часть start line запроса
- [[Идемпотентность HTTP]] — как сделать POST/PATCH идемпотентными
- [[HTTP Status Codes]] — коды ответов для каждого метода
- [[REST и API протоколы]] — REST = методы + ресурсы + статус-коды

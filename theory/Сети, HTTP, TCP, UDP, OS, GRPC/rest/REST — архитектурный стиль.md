- **REST** — это **архитектурный стиль** (набор ограничений), не протокол. Придумал Рой Филдинг в диссертации 2000 года. REST **не привязан к HTTP** — но HTTP идеально ложится
- **6 constraints**: client-server, **stateless** (сервер не хранит состояние клиента), **cacheable**, uniform interface (ресурсы + представления + self-descriptive messages + HATEOAS), layered system, code-on-demand (опц.)
- **REST-like** (Level 2) = то что **все называют** "REST": методы + ресурсы + статус-коды. **RESTful** (Level 3) = полный REST с HATEOAS — **почти никто** не делает. На собесе "RESTful API" = Level 2
- REST на HTTP/1.1 **не потому что нужен** HTTP/1.1, а потому что HTTP **идеально ложится**: методы = CRUD, URI = ресурсы, status codes = результат, headers = метаданные. HTTP/2 **тоже работает** прозрачно

---

## 6 Constraints (ограничений)

| Constraint | Суть | Зачем |
|:--|:--|:--|
| **Client-Server** | Клиент и сервер независимы | Можно менять фронт/бэк отдельно |
| **Stateless** | Каждый запрос содержит **всё** для обработки. Сервер не хранит сессию | Масштабируемость: любой сервер обработает любой запрос |
| **Cacheable** | Ответы помечаются кэшируемыми или нет | Снижение нагрузки (GET кэшируется) |
| **Uniform Interface** | Единый способ взаимодействия (ресурсы, представления, HATEOAS) | Простота, предсказуемость |
| **Layered System** | Клиент не знает общается ли с сервером напрямую или через прокси/LB | CDN, API gateway прозрачны |
| **Code-on-Demand** | Сервер может отправить клиенту код (JS) | Опциональный, почти не используется |

## Stateless — главный constraint

```
❌ Stateful: сервер помнит "юзер на шаге 3 оформления заказа"
   → sticky sessions, не масштабируется

✅ Stateless: каждый запрос содержит Authorization header, все параметры
   → любой из 100 серверов обработает → масштабируемость
```

Сессии (session_id в cookie) **формально нарушают** stateless. На практике это компромисс — состояние хранится в Redis, не на конкретном сервере.

## REST vs RESTful vs REST-like

```
REST        = теория Филдинга (6 constraints + HATEOAS)
RESTful     = полная реализация REST (Level 3, HATEOAS)  ← почти никто
REST-like   = Level 2 (методы + ресурсы + статус-коды)   ← все делают это
```

### Richardson Maturity Model

```
Level 0: POST /api {"action": "getUser"}              ← RPC, не REST
Level 1: POST /users/123                               ← ресурсы, но всё POST
Level 2: GET /users/123 → 200 OK                       ← методы + коды = "REST"
Level 3: GET /users/123 → {"_links": {"orders": "..."}} ← HATEOAS = True REST
```

**На собесе**: "расскажи про REST" = расскажи про Level 2 + упомяни что "настоящий REST" включает HATEOAS (Level 3), но на практике никто не делает.

### Почему Level 3 не взлетел

- Фронтенд **всё равно хардкодит** URL'ы (роуты в React)
- Каждый ответ раздувается ссылками → overhead
- GraphQL решает проблему гибкости **лучше**
- Нет стандарта формата ссылок (HAL, JSON-LD, Siren — зоопарк)

## Uniform Interface (4 sub-constraints)

1. **Resource identification**: каждый ресурс имеет URI (`/users/123`)
2. **Resource manipulation through representations**: JSON/XML — представление ресурса, не сам ресурс
3. **Self-descriptive messages**: Content-Type, Accept, status code — запрос/ответ самодостаточен
4. **HATEOAS**: ответ содержит ссылки на возможные действия (Level 3)

## Связь
- [[REST — дизайн API]] — практика: naming, versioning, pagination
- [[REST — HTTP версии и ограничения]] — почему HTTP, когда REST не подходит
- [[HTTP методы]] — методы = CRUD операции над ресурсами

- **Частые коды**: `InvalidArgument` (невалидный запрос), `NotFound`, `AlreadyExists`, `Unauthenticated` (нет токена), `PermissionDenied` (нет прав), `Unavailable` (retry), `DeadlineExceeded` (таймаут)
- **WithDetails()** — структурированные ошибки: `BadRequest` (какое поле), `RetryInfo` (когда повторить), `ErrorInfo` (reason + domain). Готовые типы в `errdetails`
- **Паттерн**: бизнес-логика возвращает свои ошибки → **маппинг в gRPC status на границе** (handler/interceptor), не внутри бизнес-логики

---

| Код | Когда использовать |
|:----|:-------------------|
| `InvalidArgument` | Невалидный запрос (не зависит от состояния) |
| `NotFound` | Ресурс не найден |
| `AlreadyExists` | Дубликат (создание) |
| `PermissionDenied` | Нет прав (авторизация) |
| `Unauthenticated` | Нет/невалидный токен (аутентификация) |
| `FailedPrecondition` | Нарушено условие (баланс < 0, удаление непустого) |
| `DeadlineExceeded` | Таймаут |
| `Unavailable` | Сервис временно недоступен (можно retry) |
| `Internal` | Неожиданная ошибка на сервере |

`Unknown` и `Internal` — только когда **действительно** не знаешь причину.

## Простые ошибки
```go
return nil, status.Errorf(codes.NotFound, "order %s not found", id)
```

## Структурированные ошибки — WithDetails()
```go
st := status.New(codes.InvalidArgument, "validation failed")
st, _ = st.WithDetails(&errdetails.BadRequest{
    FieldViolations: []*errdetails.BadRequest_FieldViolation{
        {Field: "email", Description: "invalid format"},
    },
})
return nil, st.Err()
```

Готовые типы в `errdetails`: `BadRequest`, `RetryInfo`, `ResourceInfo`,
`ErrorInfo` (reason + domain + metadata), `DebugInfo` (стектрейс, только dev).

## Обработка на клиенте
```go
st := status.Convert(err)
for _, detail := range st.Details() {
    switch t := detail.(type) {
    case *errdetails.BadRequest:
        // показать пользователю конкретные поля
    case *errdetails.RetryInfo:
        // подождать t.RetryDelay и повторить
    }
}
```

**Паттерн:** domain error → маппинг в gRPC status на границе сервиса (не в бизнес-логике).

## Связь
- [[gRPC — что это и когда]] — gRPC вместо HTTP status codes
- [[Interceptors]] — маппинг ошибок через interceptor
- [[HTTP Status Codes]] — аналогия: gRPC codes ↔ HTTP codes

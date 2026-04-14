- Токен в **metadata** с ключом `"authorization"` = `"Bearer <token>"`. Аналог HTTP Authorization header
- Auth interceptor: проверить публичный ли метод → достать токен из metadata → валидировать → положить claims в `ctx.Value()`
- **Unauthenticated** = нет токена / невалидный ("кто ты?"). **PermissionDenied** = токен валидный, но нет прав ("тебе нельзя")

---

## Как передаётся токен

Два способа на клиенте:
- `grpc.WithPerRPCCredentials()` — автоматически к каждому вызову
- `metadata.AppendToOutgoingContext(ctx, "authorization", token)` — руками

## Interceptor

```go
func AuthInterceptor(ctx context.Context, req any,
    info *grpc.UnaryServerInfo, handler grpc.UnaryHandler,
) (any, error) {
    if isPublic(info.FullMethod) {
        return handler(ctx, req)
    }
    md, _ := metadata.FromIncomingContext(ctx)
    token := extractBearer(md["authorization"])
    claims, err := validateJWT(token)
    if err != nil {
        return nil, status.Error(codes.Unauthenticated, "invalid token")
    }
    ctx = context.WithValue(ctx, claimsKey{}, claims)
    return handler(ctx, req)
}
```

## Публичные методы (skip auth)

Список `PublicMethods: []string{"/pkg.Auth/Login", "/grpc.health.v1.Health/Check"}` или `selector` из go-grpc-middleware.

## Связь
- [[Interceptors]] — auth interceptor в цепочке
- [[Аутентификация и авторизация]] — identification → authentication → authorization
- [[Status Codes и обработка ошибок]] — Unauthenticated vs PermissionDenied

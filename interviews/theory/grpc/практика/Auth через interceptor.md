## Как передаётся токен
Клиент кладёт токен в **metadata** с ключом `"authorization"`,
значение в формате `"Bearer <token>"`. Это стандартная конвенция gRPC,
аналогичная HTTP-заголовку Authorization.

Два способа на клиенте:
- `grpc.WithPerRPCCredentials()` — автоматически к каждому вызову
- Руками через `metadata.AppendToOutgoingContext(ctx, "authorization", token)`

## Как проверяется на сервере

Auth interceptor делает 3 вещи:
1. Проверяет, публичный ли метод (если да — пропускает)
2. Достаёт токен из metadata → валидирует (JWT, OAuth и т.д.)
3. Кладёт claims в контекст → handler получает user_id через `ctx.Value()`
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

Варианты:
- Список `PublicMethods: []string{"/pkg.Auth/Login", "/grpc.health.v1.Health/Check"}`
- **selector** из go-grpc-middleware — гибче, матчит по сервису/методу

## Unauthenticated vs PermissionDenied
- `Unauthenticated` — нет токена или токен невалидный (кто ты?)
- `PermissionDenied` — токен валидный, но нет прав (тебе нельзя)
Протокол делегирования доступа.
"Разрешаю приложению X читать мои данные в сервисе Y БЕЗ передачи пароля"

## Роли
Resource Owner: пользователь (владелец данных)
Client: приложение (хочет доступ)
Authorization Server: выдает токены (Google, GitHub)
Resource Server: API с данными

## Authorization Code Flow (самый безопасный)
1. Client редиректит на Authorization Server:
   GET /authorize?client_id=123&redirect_uri=https://app.com/callback&scope=read
2. User логинится и разрешает доступ
3. Authorization Server редиректит обратно с code:
   https://app.com/callback?code=abc456
4. Client меняет code на access_token (backend запрос):
   POST /token {code: abc456, client_secret: xxx}
5. Authorization Server возвращает:
   {access_token: "ey...", refresh_token: "...", expires_in: 3600}
6. Client использует access_token для API запросов:
   GET /api/user Authorization: Bearer ey...

## Client Credentials Flow
Machine-to-machine (сервис -> сервис, без пользователя).
POST /token {grant_type: client_credentials, client_id: 123, client_secret: xxx}

## Refresh Token
Access token истек -> используем refresh_token для получения нового access_token.
POST /token {grant_type: refresh_token, refresh_token: "..."}

## OAuth vs JWT
OAuth - протокол (КАК получить токен)
JWT - формат токена (ЧТО передается)
OAuth часто использует JWT как access_token
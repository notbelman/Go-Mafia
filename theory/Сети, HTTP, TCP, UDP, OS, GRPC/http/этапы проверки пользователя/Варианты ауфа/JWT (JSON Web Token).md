Stateless токен: header.payload.signature

## Структура
header: {"alg": "HS256", "typ": "JWT"}
payload: {"user_id": 456, "role": "admin", "exp": 1735689600}
signature: HMAC-SHA256(base64(header) + "." + base64(payload), secret)

## Проверка токена
1. Декодируем Base64 -> получаем header + payload
2. Вычисляем signature заново с секретом сервера
3. Сравниваем signature из токена с вычисленным
4. Проверяем exp (время жизни)

## Access Token + Refresh Token
Access Token: короткий (15 мин), для API запросов
Refresh Token: длинный (7 дней), для обновления access token

## Плюсы
- Stateless: не нужно хранить сессии в БД/Redis
- Масштабируемость: любой сервер проверит токен
- Работает в microservices

## Минусы
- Нельзя отозвать до истечения exp (workaround: blacklist в Redis)
- Больше размер (передается в каждом запросе)
- XSS уязвимость если хранить в localStorage
ДОКАЖИ что ты это ты.

Username: ivan@example.com
Password: ********

Проверка что ты действительно Ivan, а не кто-то притворяется.

## Методы
- Пароль (что ты знаешь)
- JWT токен (что ты имеешь - refresh token)
- Биометрия (кто ты есть)
- 2FA/MFA (комбинация факторов)

## Stateless vs Stateful
Stateless: JWT в header, сервер не хранит сессии
Stateful: session_id в cookie, сервер хранит в Redis/БД
Самый простой способ: username + password в каждом запросе.

## Flow
1. Клиент кодирует "username:password" в Base64
2. Отправляет: Authorization: Basic dXNlcjpwYXNz
3. Сервер декодирует Base64 -> проверяет username + password

## Пример
username: admin
password: secret123

Base64("admin:secret123") = "YWRtaW46c2VjcmV0MTIz"

Authorization: Basic YWRtaW46c2VjcmV0MTIz

## Плюсы
- Простота реализации
- Stateless (как JWT)

## Минусы
- Передача пароля в каждом запросе (даже в Base64 - это НЕ шифрование!)
- Обязательно только HTTPS, иначе пароль украдут
- Нет logout (браузер кеширует credentials)
- Нет токенов с истечением
- Используется редко (API keys, dev окружение)
Браузер блокирует fetch() с другого домена без разрешения сервера.

## Preflight (OPTIONS)
Браузер шлёт OPTIONS перед POST/PUT/PATCH/DELETE:
OPTIONS /api/users
Origin: https://frontend.com

Сервер отвечает:
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400  # кеш preflight на 24ч

## Simple requests (без preflight)
GET/HEAD/POST + простые headers -> preflight не нужен
Если кастомный header (Authorization) -> preflight обязателен
ЧТО тебе можно делать?

User: ivan@example.com
Role: admin
-> Можешь удалять юзеров

User: petr@example.com  
Role: viewer
-> Можешь только читать

Проверка прав доступа. Что разрешено этому пользователю.

## Модели авторизации
RBAC (Role-Based): роли (admin, moderator, user)
ABAC (Attribute-Based): атрибуты (department=sales, level>3)
ACL (Access Control List): конкретный ресурс -> список юзеров

## Где проверяется
- Gateway: проверка роли до вызова сервиса
- Сервис: проверка ownership (user_id == resource.owner_id)
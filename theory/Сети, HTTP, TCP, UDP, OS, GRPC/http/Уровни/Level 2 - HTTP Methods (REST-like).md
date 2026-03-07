Правильные HTTP методы + статус-коды. (ВСЕ МЕЙНЯТ ЭТО)

GET /users/123          -> 200 OK
POST /users             -> 201 Created
PUT /users/123          -> 200 OK
DELETE /users/123       -> 204 No Content

Клиент ЗНАЕТ структуру URL заранее (хардкодит /users/123/orders).

Это НЕ полный REST по спеке, но на практике называется "RESTful API".
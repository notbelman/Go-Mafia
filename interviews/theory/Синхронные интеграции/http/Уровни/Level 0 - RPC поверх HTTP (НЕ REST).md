Один URL, один метод (POST), action в теле запроса.

POST /api
{"action": "getUser", "userId": 123}

POST /api
{"action": "createUser", "name": "Ivan"}

Это RPC (Remote Procedure Call), не REST.
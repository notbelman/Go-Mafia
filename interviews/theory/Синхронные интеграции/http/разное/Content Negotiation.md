Клиент просит формат, сервер выбирает лучший вариант.

Accept: application/json          -> JSON response
Accept: application/xml           -> XML response
Accept-Language: ru-RU, en;q=0.9  -> русский приоритетнее
Accept-Encoding: gzip, br          -> сжатие ответа

## Server-driven vs Agent-driven
Server-driven: сервер решает по Accept headers
Agent-driven: сервер возвращает 300 Multiple Choices, клиент выбирает
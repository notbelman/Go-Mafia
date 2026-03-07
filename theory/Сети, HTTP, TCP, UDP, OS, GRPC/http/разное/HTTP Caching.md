## Cache-Control (приоритетнее Expires)
Cache-Control: public, max-age=3600  - кешировать на 1 час (CDN + браузер)
Cache-Control: private, max-age=3600 - только браузер (не CDN)
Cache-Control: no-cache              - проверять с сервером (валидация)
Cache-Control: no-store              - не кешировать вообще

## Validation (условные запросы)
ETag: "v1" (хеш ресурса)
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT

Клиент при повторном запросе:
If-None-Match: "v1" -> 304 Not Modified (ресурс не изменился)
If-Modified-Since: ... -> 304 Not Modified

## Expires (устаревший способ)
Expires: Wed, 21 Oct 2025 07:28:00 GMT
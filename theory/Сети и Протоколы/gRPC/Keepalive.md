- HTTP/2 PING-фреймы: нет ответа за Timeout → соединение мёртвое, переподключение. TCP keepalive обнаружит через **~30 мин**, gRPC keepalive — через **секунды**
- **Клиент**: `Time` (интервал пингов, мин 10s), `Timeout` (ждать PING ACK, 20s). **Сервер**: `MaxConnectionAge` (принудительно закрыть → клиент re-resolve DNS → новые поды)
- **EnforcementPolicy**: `MinTime` (мин интервал пингов клиента). Клиент пингует чаще → сервер **закрывает** соединение (`too_many_pings`)
- **Ловушка**: несогласованные настройки клиент/сервер → случайные connection reset. Деплоить: **сначала** сервер с новым MinTime, **потом** клиент

---

## Зачем

HTTP/2 соединение может "умереть" незаметно (L4 балансировщик убивает idle). TCP обнаружит через ~30 мин. gRPC keepalive — через секунды.

## Клиент

- **Time** — интервал пингов при отсутствии активности (мин 10s)
- **Timeout** — сколько ждать PING ACK (default 20s)
- **PermitWithoutStream** — пинговать даже без активных стримов

## Сервер

- **MaxConnectionIdle** — закрыть idle-соединение (GOAWAY)
- **MaxConnectionAge** — принудительно закрыть после N времени (±10% jitter). Зачем: DNS re-resolve → новые поды
- **MaxConnectionAgeGrace** — сколько ждать завершения текущих RPC после GOAWAY
- **Time / Timeout** — серверные пинги

## EnforcementPolicy (защита сервера)

- **MinTime** — минимальный интервал между пингами клиента (default 5 мин)
- **PermitWithoutStream** — разрешить пинги без стримов

Клиент пингует чаще MinTime → сервер закрывает соединение (`too_many_pings`).

## Связь
- [[Health Checking]] — keepalive = соединение, health = сервис
- [[Load Balancing]] — MaxConnectionAge для ротации подов

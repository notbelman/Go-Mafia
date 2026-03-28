- `enable.idempotence=true` — producer получает **PID** (producer ID) + нумерует каждый батч **sequence number**
- Broker хранит последние **5 sequence** на партицию. Дубль → reject (не exception, просто лог + метрика)
- Защищает **только** от дублей из-за retry самого producer. Два разных producer instance → дубли НЕ детектятся
- При рестарте producer получает **новый PID** → broker не свяжет старые и новые сообщения → нет zombie fencing без транзакций
- Overhead минимальный: +96 бит на батч (PID long + sequence int), один доп. запрос при старте

---

## Как работает

```
Producer                              Broker (partition leader)
   │                                      │
   │── send(batch, PID=7, seq=1) ────────→│ записал, запомнил seq=1
   │←── ack ──────────────────────────────│
   │                                      │
   │── send(batch, PID=7, seq=2) ────────→│ записал, запомнил seq=2
   │←── [сеть потеряла ack] ──────────────│
   │                                      │
   │── retry(batch, PID=7, seq=2) ───────→│ seq=2 уже есть → DUPLICATE
   │←── DuplicateSequenceException ───────│  (логируется, не exception)
   │                                      │
   │── send(batch, PID=7, seq=3) ────────→│ ок
```
^ip-flow

**Broker хранит последние 5 sequence** для каждого PID на каждую партицию. Поэтому `max.in.flight.requests.per.connection ≤ 5` (default = 5). ^ip-five-sequences

## Out of order sequence

```
Broker ожидает seq=3, пришёл seq=27
→ "out of order sequence" error
→ Без транзакций: можно игнорировать
→ Но это значит что messages 3-26 потеряны!
   Проверь: producer config, unclean leader election
```
^ip-out-of-order

## Поведение при failures

**Producer restart:** ^ip-producer-restart
```
Старый producer (PID=7) умер
Новый producer стартует → initTransaction() → получает PID=42
→ Если новый пошлёт то же сообщение — broker видит другой PID
→ Дубль НЕ обнаружен (разные PID = разные producer'ы)
→ Для zombie fencing нужны транзакции (transactional.id)
```

**Broker failure:** ^ip-broker-failure
```
Leader (broker 5) хранит producer state в памяти
Follower (broker 3) тоже обновляет state при репликации
→ Broker 5 упал → broker 3 стал leader → state уже в памяти ✓

Broker 5 вернулся:
→ Читает snapshot producer state с диска
→ Догоняет через репликацию → state актуален
→ Snapshot пишется: при shutdown, при создании нового сегмента,
   при crash recovery (snapshot + replay последнего сегмента)
```

## Ограничения

```
✗ producer.send() вызван дважды с тем же сообщением → дубль
  (producer не знает что записи одинаковые)
✗ Два producer instance читают один файл → оба шлют в Kafka → дубли
✗ Рестарт producer → новый PID → нет связи с предыдущим

✓ Retry из-за сетевой ошибки → дубль предотвращён
✓ Retry из-за broker error → дубль предотвращён
✓ Retry из-за producer timeout → дубль предотвращён
```
^ip-limitations

## Как включить

```
enable.idempotence=true     // всё, одна строка

Что меняется:
- Один доп. запрос при старте (получение PID)
- +96 бит overhead на каждый record batch
- Broker валидирует sequence numbers
- Порядок сообщений гарантирован даже при max.in.flight=5
- Если уже acks=all → разницы в performance нет
```
^ip-how-to-enable

## Связь
- [[Delivery Semantics]] — idempotent producer = часть exactly-once
- [[Transactions]] — транзакции расширяют idempotent producer zombie fencing'ом
- [[Producer — архитектура]] — producer flow, max.in.flight, ordering (Ch.3)
- [[Producer — надёжность]] — retries, acks, error handling (Ch.7)
- [[Replication]] — follower хранит producer state при репликации (Ch.6)
- [[Storage — сегменты и файлы]] — PID и sequence в формате батча (Ch.6)

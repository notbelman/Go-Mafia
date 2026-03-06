При обновлении отправляем событие, все подписчики инвалидируют свои кэши.
```
Сервис A: UPDATE БД → publish("user:123:updated")

Сервис B: subscribe → получил → redis.Del("user:123")
Сервис C: subscribe → получил → redis.Del("user:123")
Pod 1:    subscribe → получил → localCache.Delete("user:123")
Pod 2:    subscribe → получил → localCache.Delete("user:123")
```

- ✅ Работает в распределённой системе с несколькими подами/сервисами
- ✅ In-memory кэши на разных подах синхронизируются
- ❌ Сложнее, нужна инфраструктура (Kafka/NATS/Redis Pub/Sub)
- ❌ Eventual consistency — между событием и инвалидацией есть задержка

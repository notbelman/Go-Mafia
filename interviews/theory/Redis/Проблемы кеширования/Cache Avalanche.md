Множество ключей протухают одновременно → все запросы разом идут в БД.
```
t=0:  SET key1 TTL=5min, SET key2 TTL=5min, SET key3 TTL=5min ...
t=5min: ВСЕ ключи протухли → тысячи запросов в БД одновременно
```

**Решение: jitter (случайный разброс TTL).**
```go
baseTTL := 5 * time.Minute
jitter := time.Duration(rand.Intn(60)) * time.Second // 0-60 сек случайно

redis.Set(ctx, key, data, baseTTL+jitter)
// Ключи протухают не одновременно, а в окне 5:00–6:00
```

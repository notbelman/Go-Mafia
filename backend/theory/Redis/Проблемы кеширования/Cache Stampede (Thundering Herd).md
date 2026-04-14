Популярный ключ протух → сотни горутин одновременно идут в БД за одними и теми же данными.
```
TTL истёк на ключ "popular_item:1"

Горутина 1 → cache miss → SELECT * FROM items WHERE id=1
Горутина 2 → cache miss → SELECT * FROM items WHERE id=1
Горутина 3 → cache miss → SELECT * FROM items WHERE id=1
...
Горутина 500 → cache miss → SELECT * FROM items WHERE id=1
```

**Решение 1: singleflight** — только одна горутина идёт в БД, остальные ждут её результат.
```go
import "golang.org/x/sync/singleflight"

var group singleflight.Group

func GetItem(id string) (*Item, error) {
    cached, err := redis.Get(ctx, "item:"+id).Result()
    if err == nil {
        return unmarshal(cached), nil
    }

    // Только один запрос в БД, остальные ждут
    result, err, _ := group.Do("item:"+id, func() (interface{}, error) {
        item, err := db.GetItem(id)
        if err != nil {
            return nil, err
        }
        redis.Set(ctx, "item:"+id, marshal(item), 5*time.Minute)
        return item, nil
    })
    return result.(*Item), err
}
```

**Решение 2: раннее обновление** — обновлять ключ в фоне до истечения TTL.
```go
// TTL = 5 мин, но обновляем в фоне на 4-й минуте
if ttl < 1*time.Minute {
    go refreshCache(key) // фоновое обновление
}
return cachedData // пока отдаём старое
```
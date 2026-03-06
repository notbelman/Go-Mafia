Данные в БД обновились, но в кэше лежит старая версия.
```
t=0: кэш = {status: "active"}, БД = {status: "active"}
t=1: UPDATE БД SET status = "cancelled"
t=2: кэш = {status: "active"} ← всё ещё старое!
```

**Решение: event-based инвалидация + TTL как страховка.**
```go
func UpdateOrder(id string, status string) error {
    err := db.UpdateOrder(id, status)
    if err != nil {
        return err
    }
    redis.Del(ctx, "order:"+id) // сразу удаляем
    return nil
}

// А при записи ставим TTL на случай если Del не сработал
redis.Set(ctx, "order:"+id, data, 30*time.Second)
```

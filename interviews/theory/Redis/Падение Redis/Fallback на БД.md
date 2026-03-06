Redis упал → идём в БД, просто медленнее.
```go
func GetUser(id string) (*User, error) {
    cached, err := redis.Get(ctx, "user:"+id).Result()
    if err == nil {
        return unmarshal(cached), nil
    }

    // Redis упал или cache miss — без разницы, идём в БД
    user, err := db.GetUser(id)
    if err != nil {
        return nil, err
    }

    // Пытаемся положить в кэш, но не падаем если не получилось
    _ = redis.Set(ctx, "user:"+id, marshal(user), 5*time.Minute)
    return user, nil
}
```

Ошибку Redis **не пробрасываем наверх** — пользователь получит данные, просто медленнее.
- Single Flight — дедупликация конкурентных запросов: N горутин запрашивают один ключ → только одна идёт в БД, остальные ждут её результат
- Решает Thunder Herd problem (stampede): горячий ключ экспайрится в кэше → все ломятся в БД
- Реализация: мьютекс + map[key]*call + канал завершения в каждом call

---

## Проблема — Thunder Herd

```
Кэш: Ronaldo → данные (TTL истёк → удалён)

1000 запросов одновременно:
  → кэш miss → все 1000 идут в БД за Ronaldo
  → БД перегружена
```

Нужно: пропустить **одну** горутину в БД, остальные 999 подождут её результат. ^sf-problem

## Использование

```go
sf := NewSingleFlight()

var wg sync.WaitGroup
for i := 0; i < 1000; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        // все 1000 вызовут Do с одним ключом
        val, err := sf.Do("ronaldo", func() (interface{}, error) {
            // только ОДНА горутина выполнит это
            return db.Query("SELECT * FROM users WHERE name = 'Ronaldo'")
        })
        fmt.Println(val)
    }()
}
wg.Wait()
```

^sf-usage

## Реализация

```go
type call struct {
    val  interface{}
    err  error
    done chan struct{}
}

type SingleFlight struct {
    mu    sync.Mutex
    calls map[string]*call
}

func (sf *SingleFlight) Do(key string, fn func() (interface{}, error)) (interface{}, error) {
    sf.mu.Lock()

    // Уже есть in-flight запрос по этому ключу?
    if c, ok := sf.calls[key]; ok {
        sf.mu.Unlock()
        <-c.done           // ждём завершения первой горутины
        return c.val, c.err
    }

    // Первая горутина — создаём call
    c := &call{done: make(chan struct{})}
    sf.calls[key] = c
    sf.mu.Unlock()

    // Выполняем запрос
    go func() {
        defer func() {
            sf.mu.Lock()
            close(c.done)          // разблокируем всех ожидающих
            delete(sf.calls, key)  // убираем из map
            sf.mu.Unlock()
        }()
        c.val, c.err = fn()
    }()

    <-c.done  // первая горутина тоже ждёт (запрос в отдельной горутине)
    return c.val, c.err
}
```

^sf-impl

## Как работает

1. Горутина 1: lock → ключа нет → создаёт call → unlock → запускает fn в горутине → ждёт done
2. Горутина 2-1000: lock → ключ уже есть → unlock → ждут done
3. fn завершается → close(done) → все 1000 горутин получают результат ^sf-flow

## Почему первая горутина тоже ждёт done

Запрос `fn()` выполняется в **отдельной горутине**, а не в горутине-вызывателе. Это сделано чтобы defer с close(done) + delete сработал чисто после fn(). Поэтому первая горутина тоже блокируется на `<-c.done`. ^sf-why-wait

## Библиотека

`golang.org/x/sync/singleflight` — готовая реализация. Поддерживает `DoChan` (неблокирующий) и `Forget` (сброс ключа). ^sf-lib

## Связь
- [[Error Group]] — тоже координация группы горутин
- [[Promise и Future]] — single flight ≈ shared future (один результат для всех)
- [[Done channel]] — канал `done` в call — тот же паттерн

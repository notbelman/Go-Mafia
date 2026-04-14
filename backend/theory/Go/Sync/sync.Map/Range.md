# sync.Map: Range()

```go
m.Range(func(key, value any) bool {
    fmt.Println(key, value)
    return true  // false = остановить
})
```
^range-signature

**Особенности:** ^range-features

1. **Не блокирует map** — другие горутины могут читать/писать ^range-no-lock
2. **Нет consistent snapshot** — можешь увидеть частичные изменения ^range-no-snapshot
3. **Каждый ключ посещается 1 раз** — но значение может измениться между итерациями ^range-once
4. **O(N) всегда** — даже если вернёшь false после первого элемента ^range-on

**Что происходит внутри:**

```
1. Если amended=true:
   |
   +-- Lock(mu)
   +-- dirty --> read (promotion!)
   +-- dirty = nil
   +-- Unlock(mu)
   |
2. Итерируем по read.m (без лока)
```
^range-internals

**Можно вызывать методы map изнутри Range** — не будет deadlock. ^range-reentrant

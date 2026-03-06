После закрытия канала — установить в `nil`, чтобы отключить ветку в `select`. Это стандартный паттерн для корректной обработки нескольких каналов до их закрытия. ^disable-case-pattern

```go
for ch1 != nil || ch2 != nil {
    select {
    case v, ok := <-ch1:
        if !ok {
            ch1 = nil // Ветка отключена
            continue
        }
        process(v)
    case v, ok := <-ch2:
        if !ok {
            ch2 = nil
            continue
        }
        process(v)
    }
}
``` ^disable-case-code

Цикл продолжается пока хотя бы один канал не `nil`. Как только оба `nil` — условие `ch1 != nil || ch2 != nil` ложно, цикл завершается. ^disable-case-loop-condition

Без обнуления в `nil`: закрытый канал в `select` продолжал бы немедленно срабатывать с `ok=false` на каждой итерации — busy loop. ^disable-case-why-nil

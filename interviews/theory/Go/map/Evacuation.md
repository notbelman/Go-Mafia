[[map Flashcards - evacuation]]
**Что это:** механизм переноса данных из старых бакетов в новые при любом resize (x2 или same size). ^evac-def

**Главное:** переносит инкрементально, не за раз. При каждой write/delete переносится 1-2 бакета. ^evac-incremental

**Зачем:** избежать "stop the world" — если миллион ключей, нельзя остановиться и перенести всё разом. ^evac-why

**Как работает:**

1. При resize создаётся новый массив buckets
2. Старый сохраняется в oldbuckets
3. При каждой записи/удалении вызывается evacuate()
4. evacuate() переносит 1-2 старых бакета в новые
5. Когда всё перенесено — oldbuckets = nil

^evac-steps

```go
// при каждой write/delete
if hmap.oldbuckets != nil {
    evacuate(hmap, hmap.nevacuate)
    hmap.nevacuate++
}
```

^evac-code

**Куда едет ключ при resize x2:**

Добавляется 1 бит к маске. Ключ попадает либо в тот же индекс, либо в индекс + старый_размер. ^evac-x2-bit

```
B=2 (4 бакета) --> B=3 (8 бакетов)

bucket 2 (0b10) --> bucket 2 (0b010) если новый бит = 0
                --> bucket 6 (0b110) если новый бит = 1
```

^evac-x2-example

**Во время evacuation:**

Map работает с двумя массивами. При чтении проверяются оба — buckets и oldbuckets. ^evac-dual-read

```
oldbuckets (старые)     buckets (новые)
[0] [1] [2] [3]         [0] [1] [2] [3] [4] [5] [6] [7]
        ^                       ^
        |__ данные могут __|
            быть в любом
```

^evac-dual-diagram

**Когда evacuation завершена:**

После того как все бакеты перенесены, поле `oldbuckets` обнуляется (`oldbuckets = nil`). Поле `nevacuate` в hmap отслеживает прогресс — индекс следующего бакета для эвакуации. ^evac-completion

## Связь
- [[структура hmap]] — buckets, oldbuckets, nevacuate
- [[Resize x2]] — удвоение при load factor > 6.5
- [[Same size rehash]] — rehash при кластеризации
- [[нельзя взять адрес value]] — эвакуация = причина запрета &m[key]

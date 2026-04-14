[[map Flashcards - overflow_buckets]]
Один bucket = 8 слотов. Приходит 9-й элемент с тем же индексом — создаётся overflow bucket, это связанный список ^overflow-def

```
bucket 0: [8 элементов] --> overflow: [8] --> overflow: [8]
```

^overflow-chain

**Почему появляются:**

Коллизии. Разные ключи дают одинаковый индекс бакета. ^overflow-why

**Проблема:**

Длинные цепочки = медленный поиск (O(N)). Вместо O(1) идём по цепочке. ^overflow-problem

**Как считается индекс:**
```go
index := hash & (len(buckets) - 1)
// hash = 0b10101111, buckets = 4
// 0b10101111 & 0b11 = 3 --> bucket 3
```

^overflow-index-calc

Все ключи с одинаковым индексом идут в один бакет. Если там уже 8 — создаётся overflow. ^overflow-trigger

**Когда срабатывает same-size rehash:**

Если количество overflow-бакетов >= 2^B (где B — log2 от числа бакетов), Go запускает same-size rehash, чтобы "уплотнить" данные и избавиться от длинных цепочек. ^overflow-rehash-trigger

**Память:**

Overflow бакеты не освобождаются при `delete`. Память возвращается только после same-size rehash или GC. Это классическая причина утечки памяти в map при массовом удалении ключей. ^overflow-memory-leak

## Связь
- [[bucket (bmap)]] — структура бакета, overflow pointer
- [[Same size rehash]] — триггер: overflow >= 2^B
- [[Метод цепочек]] — теория связных списков в бакетах
- [[Память и утечки map]] — overflow бакеты не освобождаются при delete

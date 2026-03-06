[[map Flashcards - hashing]]
**Как найти bucket по ключу:**
```
hash(key, seed) = 64-битное число

                  старшие 8 бит          младшие B бит
                       ↓                      ↓
0b10110100_11010010_01110101_10010110_11101001_01101010_11110010_01110011
   └──────┘                                                          └──┘
   tophash = 0xB4                                            bucket index
   (фильтр внутри bucket)                                    (какой bucket)
```

^hash-bits-split

tophash — старшие 8 бит хеша, используется как быстрый фильтр внутри bucket. ^hash-tophash-def

bucket index — младшие B бит хеша, определяет в какой bucket попадёт ключ. ^hash-bucket-index-def

**Пример с B=2 (4 buckets):**
```
hash("foo", seed) = 0xB4...73  
                      ↑    ↑
                tophash   73 & 0b11 = 3 → bucket 3

hash("bar", seed) = 0x2F...41
                      ↑    ↑
                tophash   41 & 0b11 = 1 → bucket 1
```

^hash-example-b2

**Поиск значения по шагам:**
```go
// m["foo"]
hash := aeshash("foo", hmap.hash0)   // 1. хешируем с seed этой map
bucket := buckets[hash & 0b11]       // 2. младшие B бит → номер bucket
top := uint8(hash >> 56)             // 3. старшие 8 бит → tophash

for ; bucket != nil; bucket = bucket.overflow {
    for i := 0; i < 8; i++ {
        if bucket.tophash[i] != top { continue }  // быстрый skip
        if bucket.keys[i] == "foo" { return bucket.values[i] }
    }
}
```

^hash-lookup-code

tophash отсеивает 255/256 ключей без дорогого сравнения строк. ^hash-tophash-efficiency

## Связь
- [[bucket (bmap)]] — структура бакета, tophash массив
- [[структура hmap]] — hash0 (seed), B (количество бакетов)
- [[операции чтения и вставки]] — полный алгоритм чтения/записи
- [[Хэш-таблицы теория]] — хэш-функция и коллизии

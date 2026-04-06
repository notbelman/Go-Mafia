## Map в Go

### Теория хэш-таблиц

#### [[Хэш-таблицы теория]]
Коллизии, load factor, rehashing — общая теория

#### [[Метод открытой адресации]]
Linear/quadratic probing, tombstone, vs chaining

#### [[Метод цепочек]]
Separate chaining, linked list vs array buckets

#### [[Swiss Tables]]
Google SwissMap, SIMD lookup, metadata bytes (Go 1.24+)

---

### Внутреннее устройство Go map

#### [[структура hmap]]
hmap: count, buckets, oldbuckets, nevacuate, hash0

#### [[bucket (bmap)]]
bmap: 8 ячеек, tophash[8], keys[], values[], overflow ptr

#### [[хеширование и поиск bucket]]
hash(key) → bucket index + tophash, сравнение tophash

#### [[операции чтения и вставки]]
mapassign, mapaccess — fast path, overflow chain

#### [[Overflow buckets]]
Когда bucket переполняется, цепочка overflow buckets

#### [[Resize x2]]
Рост x2 при load factor > 6.5, incremental evacuation

#### [[Same size rehash]]
Перехеширование без роста — при большом числе overflow buckets

#### [[Evacuation]]
Пошаговая эвакуация при resize, oldbuckets, nevacuate

---

### Поведение и особенности

#### [[nil vs empty map]]
nil map паникует при записи, пустая map — нет

#### [[не потокобезопасна]]
Concurrent map write → fatal error, не data race

#### [[нельзя взять адрес value]]
Почему &m[key] не компилируется — evacuaton инвалидирует адреса

#### [[iteration order]]
Случайный порядок итерации намеренно, startBucket randomization

#### [[Итерация и мутация]]
Удаление/добавление во время range — что происходит

#### [[Память и утечки map]]
Map не возвращает память ОС, delete не освобождает buckets

#### [[Map = указатель]]
map — ссылочный тип, передача в функцию без &

#### [[HashDoS — атака на мапу]]
Коллизии хэшей → O(n) lookup, hash seed рандомизация в Go

#### [[Ключи map]]
comparable constraint, float как ключ, struct как ключ

#### [[Set в Go]]
map[T]struct{} как set, операции union/intersection

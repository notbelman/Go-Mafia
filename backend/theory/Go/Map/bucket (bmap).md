- бакет = массив **фиксированной длины 8 элементов** ^bmap-size
- layout: `[8 tophash][8 keys][8 values][overflow *bmap]` — **не** `[key,value,key,value]` ^bmap-layout
- tophash = старшие 8 бит хэша → фильтр: 255/256 ключей отсеиваются без сравнения ^bmap-tophash-filter
- keys и values отдельно: **экономия памяти** (padding) + **cache-friendly** при поиске по ключам ^bmap-kv-separate
- если ключ или значение **>128 байт** → хранится указатель, не само значение ^bmap-indirect-threshold

---
[[map Flashcards - bmap]]

**Зачем каждое поле:**
```go
type bmap struct {
    tophash  [8]uint8     // старшие 8 бит хеша каждого ключа (быстрый фильтр)
    keys     [8]KeyType   // 8 ключей подряд
    values   [8]ValueType // 8 значений подряд
    overflow *bmap        // следующий bucket если >8 элементов
}
```

## Зачем tophash?

Быстрый фильтр. При поиске "a" сначала сравниваем tophash (1 байт), а не весь ключ (дорого). 255 из 256 ключей отсеиваются без сравнения строк. ^bmap-tophash-why

Зарезервированные значения: 0 = пустая ячейка и дальше пусто, 1 = пустая ячейка, 2-5 = статусы эвакуации. ^bmap-tophash-reserved

## Зачем keys и values отдельно? (Data Oriented Design)

**Padding:** для `map[int64]int8` пары дали бы 7 байт padding на каждую. Группировка — экономит. ^bmap-padding

**Бенчмарк из лекции:**
```go
// Массив из 8 структур {key, value}
type Bucket1 struct{ Key int32; Value int32 }
var _ [8]Bucket1  // = 128 байт

// Одна структура с двумя массивами
type Bucket2 struct{ Keys [8]int32; Values [8]int32 }
var _ Bucket2     // = 72 байта  ← выигрыш из-за выравнивания
```
^bmap-padding-benchmark

**Cache-friendly:** при поиске подгружаются только ключи (не значения). Меньше мусора в кэш-линии. ^bmap-cache-friendly

## Порог 128 байт

Если ключ или значение > 128 байт — хранится указатель на данные (indirectkey/indirectvalue), а не само значение. ^bmap-indirect-why

Поэтому `[129]byte` как value потребляет столько же бакетной памяти, сколько `*[128]byte`. ^bmap-indirect-example

## Связь
- [[хеширование и поиск bucket]] — tophash и поиск внутри бакета
- [[Overflow buckets]] — overflow pointer, связный список
- [[Память и утечки map]] — порог 128 байт, указатели как решение утечек
- [[структура hmap]] — buckets указывает на массив bmap

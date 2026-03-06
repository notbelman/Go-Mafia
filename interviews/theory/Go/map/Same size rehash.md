[[map Flashcards - same_size_rehash]]
**Проблема:** ключи кластеризуются — одни бакеты переполнены, другие пустые. Поиск становится O(n). ^ssr-problem

**Решение:** перехэшировать данные с новым seed, чтобы распределить равномернее. ^ssr-solution

**Триггер:** overflow бакетов >= основных бакетов (2^B). ^ssr-trigger

**Пример кластеризации:**
```
4 бакета, 20 элементов, load factor = 5 < 6.5 (resize x2 не нужен)

bucket 0: [8] --> overflow [8] --> overflow [4]   <-- всё тут
bucket 1: [пусто]
bucket 2: [пусто]
bucket 3: [пусто]
```
^ssr-cluster-example

**Что происходит:**

1. Создаётся новый массив бакетов того же размера ^ssr-step1
2. Элементы перехэшируются с новым seed ^ssr-step2
3. Новый seed даёт другое распределение ^ssr-step3
4. Старые overflow бакеты уходят в GC ^ssr-step4

```
После rehash:
bucket 0: [6]
bucket 1: [5]
bucket 2: [4]
bucket 3: [5]
```
^ssr-after-example

B не меняется. Количество бакетов то же, но данные распределены равномерно. ^ssr-b-unchanged

**Код триггера:**
```go
// runtime/map.go
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
    if B > 15 {
        B = 15
    }
    return noverflow >= uint16(1)<<(B&15)
}
```
^ssr-code

## Связь
- [[Resize x2]] — другой вид resize (удвоение при load factor > 6.5)
- [[Evacuation]] — механизм переноса данных (общий для обоих resize)
- [[Overflow buckets]] — что триггерит same size rehash

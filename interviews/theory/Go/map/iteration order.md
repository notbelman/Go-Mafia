[[map Flashcards - iteration_order]]
Порядок итерации **рандомный by design**: ^iter-random
```go
m := map[string]int{"a": 1, "b": 2, "c": 3}

for k := range m { fmt.Print(k) }  // "bca"
for k := range m { fmt.Print(k) }  // "abc"
for k := range m { fmt.Print(k) }  // "cab"
```

**Как это сделано:**
```go
// runtime/map.go — начало итерации

// рандомное число, например 11010110
r := uintptr(rand())                  
// нижние биты → номер bucket'а (быстрый r % кол-во_buckets)
it.startBucket = r & bucketMask    
// верхние биты → с какого элемента внутри bucket'а начать  
it.offset = uint8(r >> bucketShift)
```
^iter-mechanism

**Зачем?**

Go 1.0 имел детерминированный порядок → программисты полагались на него → код ломался при смене версии/платформы. ^iter-history

С Go 1.3 порядок рандомизирован даже для маленьких map. ^iter-since-go13

**Нужен порядок?**
```go
keys := make([]string, 0, len(m))
for k := range m { keys = append(keys, k) }
slices.Sort(keys)
for _, k := range keys { fmt.Println(k, m[k]) }
```
^iter-sorted-pattern

## Связь
- [[Итерация и мутация]] — добавление/удаление во время range
- [[HashDoS — атака на мапу]] — рандомный seed тоже защита
- [[структура hmap]] — hash0 (seed)

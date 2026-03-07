[[map Flashcards - resize_x2]]
Когда map удваивает количество бакетов.

**Условие:**

load factor > 6.5 (в среднем больше 6.5 элементов на бакет) ^rx2-trigger

**Пример:**
```
B = 2 --> 4 бакета --> resize при count > 26 (26/4 = 6.5)
B = 3 --> 8 бакетов --> resize при count > 52
```
^rx2-example-b

**Что происходит:**
```
B = 2 (4 бакета)          B = 3 (8 бакетов)
[0] [1] [2] [3]     -->   [0] [1] [2] [3] [4] [5] [6] [7]
```

B увеличивается на 1, количество бакетов удваивается. ^rx2-b-increment

Данные переносятся через evacuation (постепенно, не за раз). ^rx2-evacuation

**Почему 6.5:**

Компромисс. Меньше — быстрый поиск, но много пустых бакетов. Больше — экономим память, но длинные цепочки. ^rx2-why-65

```go
// runtime/map.go
func overLoadFactor(count int, B uint8) bool {
    return count > 8 && 
           uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
// loadFactorNum = 13, loadFactorDen = 2 --> 13/2 = 6.5
```
^rx2-code

## Связь
- [[Evacuation]] — как происходит перенос данных
- [[Same size rehash]] — другой вид resize (без удвоения)
- [[структура hmap]] — поле B (лог2 бакетов)
- [[Хэш-таблицы теория]] — load factor

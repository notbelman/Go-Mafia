#flashcards/map/hmap_struct

Чем map-переменная отличается от slice/string на уровне реализации?
?
![[структура hmap#^hmap-is-pointer]]

Перечисли ключевые поля hmap и назначение каждого
?
![[структура hmap#^hmap-struct-fields]]

Почему B (количество бакетов) хранится в логарифмической форме?
?
![[структура hmap#^hmap-b-log2]]

Зачем в hmap поле hash0? Что будет без него?
?
![[структура hmap#^hmap-hash0-dos]]

Как hmap.flags используется для детекции concurrent write?
?
![[структура hmap#^hmap-flags-bits]]

Зачем hmap нужны два поля: buckets и oldbuckets?
?
![[структура hmap#^hmap-oldbuckets]]

Что такое nevacuate в hmap?
?
![[структура hmap#^hmap-oldbuckets]]

`make(map[int]int, N)` — это точный cap как у slice?
?
![[структура hmap#^hmap-make-not-exact-cap]]

make(map, 20) — сколько бакетов будет выделено и какой максимальный размер?
?
![[структура hmap#^hmap-make-table]]

make(map, 28) — сколько бакетов?
?
![[структура hmap#^hmap-make-table]]

Сколько бакетов у map без указания размера?
?
![[структура hmap#^hmap-default-one-bucket]]

Почему резервирование через make(map, N) на порядок быстрее чем без него?
?
![[структура hmap#^hmap-make-perf]]

Что выведет этот код?
```go
m1 := make(map[int]int)
m2 := m1
m2[1] = 42
fmt.Println(m1[1])
```
?
`42` — map-переменная это указатель на hmap. m1 и m2 указывают на одну структуру, изменение через любую переменную видно через другую.
![[структура hmap#^hmap-is-pointer]]

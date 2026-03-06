#flashcards/sync-map/range

Четыре особенности Range() в sync.Map.
?
![[Range#^range-features]]

Range() блокирует map? Могут ли другие горутины читать/писать во время Range?
?
![[Range#^range-no-lock]]

Гарантирует ли Range() consistent snapshot всей map?
?
![[Range#^range-no-snapshot]]

Сложность Range() — O(N) всегда, даже если вернёшь false после первого элемента. Почему?
?
![[Range#^range-on]]

Что происходит внутри Range() если amended=true?
?
![[Range#^range-internals]]

Можно ли вызывать методы map изнутри Range? Будет ли deadlock?
?
![[Range#^range-reentrant]]

Что выведет этот код (порядок не важен)?
```go
var m sync.Map
m.Store("a", 1)
m.Store("b", 2)
m.Store("c", 3)
m.Range(func(k, v any) bool {
    fmt.Println(k, v)
    return true
})
```
?
Выведет три строки: `a 1`, `b 2`, `c 3` (порядок не определён). Range обходит все живые ключи. Если amended=true — сначала promotion, потом итерация по read.m.
![[Range#^range-internals]]

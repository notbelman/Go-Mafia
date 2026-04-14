#flashcards/map/concurrent

Что произойдёт при одновременном чтении и записи в map из разных горутин?
?
![[не потокобезопасна#^mt-fatal-not-panic]]

Почему concurrent map read/write — это fatal error, а не panic? Можно ли его recover?
?
![[не потокобезопасна#^mt-fatal-not-panic]]

Как runtime детектирует одновременную запись в map?
?
![[не потокобезопасна#^mt-flags-detection]]

Какое поле hmap используется для детекции concurrent write?
?
![[не потокобезопасна#^mt-flags-detection]]

В чём разница между sync.Mutex и sync.RWMutex при защите map?
?
![[не потокобезопасна]]

Когда sync.Map быстрее чем map + RWMutex?
?
![[не потокобезопасна#^mt-syncmap-when]]

Что выведет этот код?
```go
m := make(map[int]int)
go func() {
    for { m[1] = 1 }
}()
go func() {
    for { _ = m[1] }
}()
select {}
```
?
`fatal error: concurrent map read and map write` — runtime ловит через hmap.flags, это не panic и recover не поможет.
![[не потокобезопасна#^mt-fatal-not-panic]]

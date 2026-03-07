#flashcards/sync-map/walkthrough

Первый Store на пустую sync.Map. Что происходит внутри? Почему?
?
![[Пример_ебнутый#^example-step1]]

После двух Store и трёх Load. Почему произошёл promotion? Каковы условия?
?
![[Пример_ебнутый#^example-step3]]

После первого promotion делаем Store("c", 3). dirty == nil. Опиши шаги.
?
![[Пример_ебнутый#^example-step4]]

Delete("b") когда b есть в read. Что произошло с entry в dirty?
?
![[Пример_ебнутый#^example-step5]]

После второго promotion (с nil в read) делаем Store("d", 4). Что произошло с b?
?
![[Пример_ебнутый#^example-step7]]

Store("b", 5) когда b = expunged в read. Опиши три шага.
?
![[Пример_ебнутый#^example-step8]]

Что произойдёт с состоянием sync.Map после этой последовательности?
```
1. var m sync.Map
2. m.Store("a", 1)
3. m.Store("b", 2)
4. m.Load("a")
5. m.Load("b")
6. m.Load("a")  ← 3-й промах
```
?
После шага 6: misses(3) >= len(dirty)(2) → promotion. read: {a:*1, b:*2}, amended=false, dirty=nil, misses=0.
![[Пример_ебнутый#^example-first-promotion]]

Полная трассировка: после Store("d",4) что именно в read и dirty? Где b?
?
![[Пример_ебнутый#^example-second-promotion]]

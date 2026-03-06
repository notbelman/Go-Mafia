#flashcards/slice/gc_and_slice

При каком условии GC может собрать underlying array слайса, даже если дескриптор ещё жив?
?
![[GC и slice#^gc-slice-collect-rule]]
![[GC и slice#^469a4f]]


Что удерживает underlying array слайса от сбора GC?
?
![[GC и slice#^gc-slice-keep-rule]]
![[GC и slice#^2350b2]]

Что делает `runtime.KeepAlive(s)` и когда его использовать?
?
![[GC и slice#^gc-keepalive-what]]
![[GC и slice#^090ef8]]

Почему `runtime.KeepAlive` должен стоять ПОСЛЕ использования данных, а не до вызова GC?
?
![[GC и slice#^gc-keepalive-placement]]
![[GC и slice#^090ef8]]

Что выведет этот код — и что произойдёт с памятью?
```go
s := make([]int, 1<<27)
runtime.GC()
fmt.Println(len(s))
```
?
Выведет `134217728`. Но underlying array (~1 ГБ) уже может быть освобождён: GC видит, что к `s[i]` никто не обращается, только к `len`. Дескриптор жив, данные — нет.
![[GC и slice#^gc-slice-collect-rule]]

Что выведет этот код — освободит ли GC массив?
```go
s := make([]int, 1<<27)
runtime.GC()
fmt.Println(s[0])
```
?
Выведет `0`. Обращение к `s[0]` удерживает underlying array — GC не соберёт.
![[GC и slice#^gc-slice-keep-rule]]

В чём проблема стандартной реаллокации слайса в realtime-системах?
?
![[GC и slice#^gc-incr-problem]]

Как работает инкрементальное копирование слайса? Опиши алгоритм.
?
![[GC и slice#^gc-incr-algorithm]]
![[GC и slice#^c5c535]]

Какой трейдофф у инкрементального копирования по сравнению со стандартной реаллокацией?
?
![[GC и slice#^gc-incr-tradeoff]]

Какова worst-case сложность вставки при инкрементальном копировании vs стандартном append?
?
![[GC и slice#^gc-incr-tradeoff]]

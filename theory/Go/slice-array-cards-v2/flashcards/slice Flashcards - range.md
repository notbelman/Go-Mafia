#flashcards/slice/range

Что именно range копирует перед стартом цикла и что из этого следует?
?
![[Range подводные камни#^range-copy-once]]

Почему `append` внутри `for _, v := range s` не создаёт бесконечный цикл?
?
![[Range подводные камни#^range-append-not-infinite]]

Что выведет этот код?
```go
s := []int{1, 2, 3}
for _, v := range s {
    s = append(s, v*10)
}
fmt.Println(len(s), s)
```
?
`6 [1 2 3 10 20 30]` — ровно 3 итерации, потому что range скопировал дескриптор с len=3 до старта.
![[Range подводные камни#^range-copy-once]]

Чем `for _, v := range s { s = append(...) }` отличается от `for i := 0; i < len(s); i++` с append внутри?
?
![[Range подводные камни#^range-vs-classic-loop]]

Модифицирует ли этот код массив?
```go
a := [3]int{1, 2, 3}
for _, v := range a {
    v += 50
}
fmt.Println(a)
```
?
`[1 2 3]` — не изменился. `v` — копия элемента, модификация копии не влияет на оригинал.
![[Range подводные камни#^range-value-copy]]

Какую проблему с горутинами решил Go 1.22 в range?
?
![[Range подводные камни#^range-go122-fix]]

Что происходило с loop variable в range до Go 1.22 и почему это ломало горутины в замыканиях?
?
![[Range подводные камни#^range-go122-before]]

Почему модификация по индексу `a[i]` быстрее, чем через срез указателей `[]*T`?
?
![[Range подводные камни#^range-modify-index]]

Что происходит с памятью при `for _, v := range bigArr` где `bigArr [1_000_000]int`?
?
![[Range подводные камни#^range-array-copy-overhead]]

Как избежать копирования массива при range? Два способа.
?
![[Range подводные камни#^range-array-copy-fix]]

Что выведет этот код и почему?
```go
bigArr := [4]int{1, 2, 3, 4}
for i, v := range bigArr {
    bigArr[i] = v * 10
}
fmt.Println(bigArr)
```
?
`[10 20 30 40]` — range скопировал массив один раз, `v` берётся из копии (оригинальные значения), но `bigArr[i] =` пишет в оригинал.
![[Range подводные камни#^range-copy-once]]

В чём смысл loop unrolling и когда его применять?
?
![[Range подводные камни#^range-loop-unrolling]]

#flashcards/sync-map/len

Почему sync.Map не имеет метода Len()?
?
![[Len#^len-problem]]
![[Len#^len-complexity]]

Как получить количество элементов в sync.Map?
?
![[Len#^len-workaround]]

Какова сложность workaround для подсчёта элементов? Есть ли побочные эффекты?
?
![[Len#^len-workaround-detail]]

Что выведет этот код?
```go
var m sync.Map
m.Store("a", 1)
m.Store("b", 2)
m.Delete("a")
count := 0
m.Range(func(_, _ any) bool {
    count++
    return true
})
fmt.Println(count)
```
?
`1` — Delete помечает "a" как nil. Range пропускает nil/expunged записи, считает только живые. b — единственный живой ключ.
![[Len#^len-workaround]]

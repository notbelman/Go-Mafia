#flashcards/str/interning

Почему Go автоматически интернирует строковые литералы, но не runtime-строки?
?
![[interning#^interning-compiler-text-segment]]

Что произойдёт с памятью, если в файле 5000 раз встречается строка "the" и ты не используешь interning?
?
![[interning#^interning-problem]]

Как реализовать ручной interning через map? Что происходит при повторном обращении к уже интернированной строке?
?
![[interning#^interning-manual-map]]

Что выведет этот код?
```go
s1 := "hello"
s2 := "hello"
const c = "hello"
fmt.Println(unsafe.StringData(s1) == unsafe.StringData(s2))
fmt.Println(unsafe.StringData(s1) == unsafe.StringData(c))
```
?
`true` и `true` — все три строки живут в text segment (read-only). Литерал де-факто константа, потому что строки в Go immutable. Компилятор дедуплицирует одинаковые литералы.
![[interning#^interning-compiler-text-segment]]

С какой версии Go появился пакет `unique` и что он предоставляет?
?
![[interning#^interning-unique-internals]]

Как работает сравнение через `unique.Handle`? Почему оно O(1)?
?
![[interning#^interning-unique-internals]]

Что внутри пакета `unique`: как устроен глобальный пул, goroutine-safe ли он, и работает ли только со строками?
?
![[interning#^interning-unique-internals]]

Почему сравнение через `unique.Handle` быстрее обычного сравнения строк? Что показывает бенчмарк?
?
![[interning#^interning-unique-bench]]

Когда имеет смысл применять interning? Назови конкретные сценарии.
?
![[interning#^interning-use-cases]]

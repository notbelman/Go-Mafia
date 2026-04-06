#flashcards/str/builder

Почему конкатенация через `+` в цикле — O(n²)?
?
![[strings_Builder#^builder-why-loop-quadratic]]

Сколько аллокаций делает `strings.Builder` при 10000 итерациях без `Grow`? Почему не 10000?
?
![[strings_Builder#^builder-internals-slice]]

Что внутри `strings.Builder`? Как реализован `WriteString`?
?
![[strings_Builder#^builder-internals-slice]]

Как реализован `Builder.String()`? Почему он не копирует данные?
?
![[strings_Builder#^builder-string-unsafe]]

Что делает `Builder.Grow(n)` и какой эффект это даёт на количество аллокаций?
?
![[strings_Builder#^builder-grow]]

Что выведет этот код?
```go
var b1 strings.Builder
b1.WriteString("hello")
b2 := b1
b2.WriteString(" world")
fmt.Println(b1.String())
fmt.Println(b2.String())
```
?
Panic при `b2.WriteString(" world")` — копирование Builder после использования запрещено. Два Builder шарят один `[]byte`, что детектируется через `noCopy` механизм.
![[strings_Builder#^builder-copy-panic]]

Почему нельзя копировать `strings.Builder` после первой записи? Что произойдёт?
?
![[strings_Builder#^builder-copy-panic]]

Сравни скорость `fmt.Sprintf`, `strings.Join` и `+` для конкатенации фиксированного набора строк. Почему именно такой порядок?
?
![[strings_Builder#^builder-plus-optimization]]

Как компилятор оптимизирует цепочку `+` для фиксированных строк? Сколько аллокаций?
?
![[strings_Builder#^builder-plus-optimization]]

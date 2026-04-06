#flashcards/slice/array_basics

Почему `[3]int` и `[4]int` — разные типы в Go? Что это означает практически?
?
![[Array основы#^array-different-types]]

Как Go вычисляет адрес элемента массива по индексу?
?
![[Array основы#^array-index-formula]]

Почему `len()` массива — константа на compile time?
?
![[Array основы#^array-size-in-type]]

Что выведет этот код?
```go
n := 5
var a [n]int
fmt.Println(a[0])
```
?
Не скомпилируется — размер массива должен быть константой, переменная `n` не подходит.
![[Array основы#^array-size-in-type]]

Что произойдёт: индекс-константа за границей vs индекс-переменная за границей?
?
![[Array основы#^array-bound-check-rules]]

Что выведет этот код?
```go
a := [3]int{1, 2, 3}
i := 5
fmt.Println(a[i])
```
?
Паника в runtime: `runtime error: index out of range [5] with length 3`. Переменный индекс — проверка в runtime, не compile time.
![[Array основы#^array-bound-check-rules]]

Поддерживает ли Go отрицательные индексы в стиле Python (`a[-1]`)?
?
![[Array основы#^array-no-negative-index]]

Что содержит `var a [10]int` сразу после объявления? Можно ли проверить его на nil?
?
![[Array основы#^array-zero-init]]

Чем сравнение массивов отличается от сравнения слайсов?
?
![[Array основы#^array-comparable]]

Почему для массивов недоступны `make` и `append`?
?
![[Array основы#^array-no-make-append]]

Что выведет этот код?
```go
a := [3]int{1, 2, 3}
b := a
b[0] = 99
fmt.Println(a[0], b[0])
```
?
`1 99` — массивы копируются по значению. `b` — независимая копия, изменение `b[0]` не затрагивает `a`.
![[Array основы#^array-different-types]]

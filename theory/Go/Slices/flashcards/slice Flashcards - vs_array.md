#flashcards/slice/vs_array

Являются ли `[3]int` и `[4]int` одним типом в Go?
?
![[Slice vs Array#^array-size-in-type]]

Чем отличается `len()` для массива vs слайса с точки зрения компилятора?
?
![[Slice vs Array#^array-len-compile-time]]

Является ли массив в Go указателем?
?
![[Slice vs Array#^array-not-pointer]]

Как вычисляется адрес элемента массива по индексу?
?
![[Slice vs Array#^array-index-access]]

Можно ли сравнивать массивы через `==`? А слайсы?
?
![[Slice vs Array#^array-comparable]] + ![[Slice vs Array#^slice-not-comparable]]

Что произойдёт при передаче массива `[1_000_000]int` в функцию?
?
![[Slice vs Array#^array-pass-copy]]

Что копируется при передаче слайса в функцию? Сколько байт?
?
![[Slice vs Array#^slice-pass-header]]

Что выведет этот код?
```go
func modify(arr [3]int) { arr[0] = 999 }
a := [3]int{1, 2, 3}
modify(a)
fmt.Println(a[0])
```
?
`1` — массив передаётся по значению, функция получает независимую копию.
![[Slice vs Array#^array-pass-copy]]

Что выведет этот код?
```go
func modify(s []int) { s[0] = 999 }
s := []int{1, 2, 3}
modify(s)
fmt.Println(s[0])
```
?
`999` — слайс передаёт header с ptr на общий underlying array, модификация видна снаружи.
![[Slice vs Array#^slice-pass-header]]

Каков zero value массива? Каков zero value слайса?
?
![[Slice vs Array#^array-zero-value]] + ![[Slice vs Array#^slice-zero-value]]

Для каких типов данных работают `make` и `append`?
?
![[Slice vs Array#^slice-make-append-only]]

Назови три сценария где массив предпочтительнее слайса.
?
![[Slice vs Array#^array-vs-slice-when]]

Что не скомпилируется и почему?
```go
var a [3]int
var b [4]int
a = b
```
?
Ошибка компиляции: `[3]int` и `[4]int` — разные типы. Размер является частью типа массива.
![[Slice vs Array#^array-size-in-type]]

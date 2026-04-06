#flashcards/generics/type_inference

Что такое type inference в дженериках Go и из чего компилятор выводит тип?
?
![[Type_inference_и_параметры_типов#^ti-definition]]

Почему type inference не работает для generic-структур, даже если тип очевиден из поля?
?
![[Type_inference_и_параметры_типов#^ti-no-structs]]
![[Type_inference_и_параметры_типов#^ti-struct-obvious]]

Почему этот код не компилируется, хотя тип слева очевиден?
```go
func create[T any]() *T {
    var v T
    return &v
}
var p *int = create()
```
?
Компилятор выводит типы только из аргументов вызова, но не из контекста присваивания. `create()` не имеет аргументов с generic-типом — нечего вывести, нужно `create[int]()`.
![[Type_inference_и_параметры_типов#^ti-no-left-side]]

В чём разница между типами-параметрами и типами-аргументами в дженериках?
?
![[Type_inference_и_параметры_типов#^ti-params-vs-args]]

Что выведет этот код и почему?
```go
func process1[T comparable, K any, E int](a T, b K) E { var e E; return e }
result := process1[int]("hello", 3.14)
_ = result
```
?
Не скомпилируется — `int` подставится в `T` (первый параметр), а `E` остаётся невыведенным. Пропуск работает только с начала (префикс). Чтобы указать `E`, нужно переставить его первым.
![[Type_inference_и_параметры_типов#^ti-prefix-rule]]

Объясни правило пропуска типов-аргументов при вызове generic-функции с несколькими type params.
?
![[Type_inference_и_параметры_типов#^ti-prefix-only]]
![[Type_inference_и_параметры_типов#^ti-prefix-rule]]

Что выведет этот код?
```go
func print[T any](v T) { fmt.Println(v) }

print(100)
print("hello")
```
?
`100` и `hello` — type inference выводит `int` и `string` из аргументов. Явное `print[int](100)` не требуется.
![[Type_inference_и_параметры_типов#^ti-definition]]

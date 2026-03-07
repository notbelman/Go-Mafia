#flashcards/unsafe/pointer_uintptr

Что такое unsafe.Pointer? Какой аналог в C?
?
![[unsafe_Pointer_и_uintptr#^up-what-is]]

Чем uintptr отличается от unsafe.Pointer с точки зрения GC?
?
![[unsafe_Pointer_и_uintptr#^up-uintptr-not-ptr]]

Почему нельзя напрямую кастить *int в *float64? Как обойти через unsafe?
?
![[unsafe_Pointer_и_uintptr#^up-chain]]

Почему хранить адрес в uintptr между строками кода опасно? Что может произойти?
?
![[unsafe_Pointer_и_uintptr#^up-gc-danger]]

Можно ли разыменовать unsafe.Pointer напрямую?
?
![[unsafe_Pointer_и_uintptr#^up-what-is]]

Какие операции разрешены для uintptr, но запрещены для unsafe.Pointer?
?
![[unsafe_Pointer_и_uintptr#^up-uintptr-not-ptr]]

Что выведет этот код?
```go
var i int64 = 1
f := *(*float64)(unsafe.Pointer(&i))
fmt.Println(f)
```
?
`5e-324` (или похожее маленькое число) — биты `int64(1)` реинтерпретируются как IEEE 754 float64. Это не конверсия значения, а реинтерпретация сырых байт.
![[unsafe_Pointer_и_uintptr#^up-chain]]

Что выведет этот код — валиден ли он?
```go
type A struct{ x int }
a := A{x: 42}
addr := uintptr(unsafe.Pointer(&a))
runtime.GC()
p := unsafe.Pointer(addr)
fmt.Println(*(*A)(p))
```
?
Поведение **undefined**. После `runtime.GC()` объект `a` мог переместиться (или быть собран). `addr` хранит старый адрес — dangling pointer. Компилятор не видит ссылки на `a` и может собрать его.
![[unsafe_Pointer_и_uintptr#^up-gc-danger]]

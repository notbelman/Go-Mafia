#flashcards/str/unsafe_string_byte

Какие функции Go 1.20+ используются для unsafe конверсии между string и []byte? Что они дают?
?
![[Unsafe string to byte#^unsafe-modern-api]]
![[Unsafe string to byte#^unsafe-no-alloc]]
![[Unsafe string to byte#^unsafe-perf]]

Во сколько раз быстрее unsafe конверсия string ↔ []byte по сравнению с обычной? Сколько аллокаций?
?
![[Unsafe string to byte#^unsafe-perf]]

Что произойдёт если изменить []byte после unsafe конверсии в string?
?
![[Unsafe string to byte#^unsafe-mutation-example]]
![[Unsafe string to byte#^unsafe-consequences]]

Три последствия нарушения иммутабельности строки через unsafe. Назови их.
?
![[Unsafe string to byte#^unsafe-consequences]]

Три условия когда unsafe конверсию string ↔ []byte можно использовать безопасно.
?
![[Unsafe string to byte#^unsafe-safe-condition-1]]
![[Unsafe string to byte#^unsafe-safe-condition-2]]
![[Unsafe string to byte#^unsafe-safe-condition-3]]

Как выглядит deprecated способ конверсии []byte → string через unsafe? Почему он работает?
?
![[Unsafe string to byte#^unsafe-deprecated-cast]]
![[Unsafe string to byte#^unsafe-why-works]]

Чем deprecated каст `*(*string)(unsafe.Pointer(&b))` лучше и хуже `unsafe.String`?
?
![[Unsafe string to byte#^unsafe-deprecated-risk]]

Что произойдёт при попытке изменить строковый литерал через unsafe.Slice?
?
![[Unsafe string to byte#^unsafe-literal-sigbus]]
![[Unsafe string to byte#^unsafe-text-segment]]

Почему строковые литералы нельзя изменить через unsafe? Где они живут?
?
![[Unsafe string to byte#^unsafe-text-segment]]

Какая стандартная библиотека Go использует unsafe конверсию string ↔ []byte внутри?
?
![[Unsafe string to byte#^unsafe-builder-internal]]

Что выведет этот код?
```go
b := []byte("hello")
s := unsafe.String(&b[0], len(b))
b[0] = 'H'
fmt.Println(s)
```
?
`Hello` — s и b шарят одну память. Изменение b[0] меняет и строку s, потому что нет копирования. Иммутабельность нарушена.
![[Unsafe string to byte#^unsafe-mutation-example]]

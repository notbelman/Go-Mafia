#flashcards/str/byte_conversion

Всегда ли `[]byte(s)` и `string(b)` создают аллокацию? Почему?
?
![[byte конверсия#^conv-always-alloc]]
![[byte конверсия#^conv-immutability-reason]]

Сколько байт в памяти займёт этот код? Почему?
```go
s := "hello"
b := []byte(s)
s2 := string(b)
```
?
![[byte конверсия#^conv-triple-cost]]
![[byte конверсия#^conv-triple-example]]

Почему нельзя напрямую скастовать `[]byte` в `[]rune`? Сколько аллокаций нужно?
?
![[byte конверсия#^conv-no-direct-rune]]
![[byte конверсия#^conv-rune-two-allocs]]

Как сделать конверсию `[]byte → []rune` за одну аллокацию?
?
![[byte конверсия#^conv-manual-rune]]

Назови 4 случая когда компилятор Go убирает аллокацию при конверсии string ↔ []byte.
?
![[byte конверсия#^conv-compiler-opts]]
![[byte конверсия#^conv-compiler-opts-examples]]

Почему `_ = m[string(b)]` не создаёт аллокацию?
?
![[byte конверсия#^conv-compiler-reasoning]]
![[byte конверсия#^conv-compiler-opts-examples]]

Что выведет?
```go
s := "hello"
b := []byte(s)
b[0] = 'H'
fmt.Println(s)
fmt.Println(string(b))
```
?
```
hello
Hello
```
`[]byte(s)` создаёт копию. Изменение `b` не влияет на оригинальную строку `s` — иммутабельность обеспечивается копированием.
![[byte конверсия#^conv-immutability-example]]

Почему `for i, b := range []byte(s)` не создаёт аллокацию?
?
![[byte конверсия#^conv-compiler-reasoning]]

Во сколько раз unsafe конверсия быстрее обычной `[]byte(s)`?
?
![[byte конверсия#^conv-unsafe-perf]]

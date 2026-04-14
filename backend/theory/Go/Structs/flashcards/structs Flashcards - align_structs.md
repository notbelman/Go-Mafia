#flashcards/structs/align_structs

Как определяется требуемое выравнивание структуры в Go?
?
![[Выравнивание структур#^align-rule-max]]

Меняет ли компилятор Go порядок полей структуры для оптимизации?
?
![[Выравнивание структур#^align-compiler-no-reorder]]

Чем заполняется padding в структурах Go — нулями или мусором?
?
![[Выравнивание структур#^align-padding-zeros]]

Как определяется выравнивание массива `[123]byte`? А `[123]int16`?
?
![[Выравнивание структур#^align-array-rule]]

Как определяется выравнивание вложенной структуры?
?
![[Выравнивание структур#^align-nested-unpack]]

Что выведет этот код?
```go
type Data1 struct {
    A bool
    B int32
    C bool
}
type Data2 struct {
    B int32
    A bool
    C bool
}
fmt.Println(unsafe.Sizeof(Data1{}), unsafe.Sizeof(Data2{}))
```
?
`12 8` — Data1: bool+3pad+int32+bool+3pad=12. Data2: int32+bool+bool+2pad=8. Компилятор не меняет порядок, padding зависит от нас.
![[Выравнивание структур#^align-field-order-example]]

Почему размер структуры всегда кратен её выравниванию? Что будет если последнее поле — bool в структуре с выравниванием 4?
?
![[Выравнивание структур#^align-trailing-padding]]

Какое выравнивание у этой структуры и почему?
```go
type X struct {
    A bool
    B [123]byte
    C bool
}
```
?
![[Выравнивание структур#^align-array-example]]

Какое выравнивание у `Outer` и почему не 16?
```go
type Inner struct {
    Ptr unsafe.Pointer
    Size int
}
type Outer struct {
    Flag bool
    Data Inner
}
```
?
![[Выравнивание структур#^align-nested-example]]

На 32-битной машине, на сколько байт выровнен int64?
?
![[Выравнивание структур#^align-32bit]]

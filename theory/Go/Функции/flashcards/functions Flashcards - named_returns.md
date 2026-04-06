#flashcards/functions/named_returns

Что такое именованные возвращаемые значения с точки зрения компилятора?
?
![[Именованные_возвращаемые_значения#^named-ret-is-decl]]

В каких двух случаях Go community рекомендует использовать именованные возвращаемые значения?
?
![[Именованные_возвращаемые_значения#^named-ret-when]]

Какие zero value получают именованные переменные результата в момент вызова функции?
?
![[Именованные_возвращаемые_значения#^named-ret-zero-value]]

Что вернёт голый `return` в функции с именованными возвращаемыми?
?
![[Именованные_возвращаемые_значения#^named-ret-zero-value]]

Что выведет этот код?
```go
func f() (x int, err error) {
    return
}
func main() {
    v, e := f()
    fmt.Println(v, e)
}
```
?
`0 <nil>` — голый return возвращает zero value: `x = 0`, `err = nil`.
![[Именованные_возвращаемые_значения#^named-ret-zero-value]]

Почему плохо смешивать голый и явный `return` в одной функции?
?
![[Именованные_возвращаемые_значения#^named-ret-dont-mix]]

Правило об именовании: если один результат именован, что обязательно для остальных?
?
![[Именованные_возвращаемые_значения#^named-ret-all-or-nothing]]

Валиден ли такой прототип? `func Write(_ []byte) (n int, err error)`
?
![[Именованные_возвращаемые_значения#^named-ret-all-or-blank]]

Когда именованные возвращаемые значения особенно полезны помимо читаемости? (hint: defer)
?
![[Именованные_возвращаемые_значения#^named-ret-when]]

Что выведет этот код?
```go
func divide(a, b float64) (result float64, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return
    }
    result = a / b
    return
}
func main() {
    r, e := divide(10, 0)
    fmt.Println(r, e)
}
```
?
`0 division by zero` — голый return при b==0 возвращает result=0.0 (zero value, не изменялся) и err=ошибка.
![[Именованные_возвращаемые_значения#^named-ret-zero-value]]

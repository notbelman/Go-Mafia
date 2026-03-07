#flashcards/generics/constraints_methods_fields

Что из двух работает в Go constraints: ограничение по методу или ограничение по полю структуры?
?
![[Constraints на методы и поля#^cm-method-works]]

Почему нельзя обратиться к полю через generic, даже если все типы в union имеют это поле?
?
![[Constraints на методы и поля#^cm-why-no-field]]

Есть `Data1{Value int}` и `Data2{Value int}`. Как написать generic-функцию `getValue`, которая читает `Value` из обоих?
?
![[Constraints на методы и поля#^cm-getter-solution]]

Что произойдёт при компиляции?
```go
type Data1 struct{ Value int }
type Data2 struct{ Value int }

func getValue[T Data1 | Data2](v T) int {
    return v.Value
}
```
?
Ошибка компиляции — компилятор не разрешает обращение к полям через union constraint, даже если все типы имеют поле `Value`.
![[Constraints на методы и поля#^cm-field-compile-error]]

Что выведет этот код?
```go
type Data1 struct{ Value int }
func (d Data1) GetValue() int { return d.Value }

type Data2 struct{ Value int }
func (d Data2) GetValue() int { return d.Value }

func getValue[T interface{ GetValue() int }](v T) int {
    return v.GetValue()
}

fmt.Println(getValue(Data1{Value: 42}))
fmt.Println(getValue(Data2{Value: 7}))
```
?
`42` и `7` — геттер-хак работает: метод `GetValue()` описан в constraint, компилятор принимает оба типа.
![[Constraints на методы и поля#^cm-getter-solution]]

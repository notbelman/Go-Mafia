#flashcards/interfaces/what_is_interface

Что такое интерфейс в Go? Как его можно определить одной фразой?
?
![[Что такое интерфейс#^iface-contract]]

Как тип реализует интерфейс в Go? Нужно ли явно указывать что-то кроме методов?
?
![[Что такое интерфейс#^iface-implicit]]

Что выведет этот код?
```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
type MyWriter struct{}
func (w MyWriter) Write(p []byte) (n int, err error) {
    fmt.Println("wrote")
    return len(p), nil
}
var w Writer = MyWriter{}
w.Write([]byte("hi"))
```
?
`wrote` — MyWriter реализует Writer неявно: нет ключевого слова `implements`, достаточно наличия метода с совпадающей сигнатурой.
![[Что такое интерфейс#^iface-no-implements]]

Сколько интерфейсов может реализовать один тип? Сколько типов могут реализовать один интерфейс?
?
![[Что такое интерфейс#^iface-many]]

Что такое композиция интерфейсов? Приведи пример из стандартной библиотеки Go.
?
![[Что такое интерфейс#^iface-composition]]

Если тип реализует Reader и Writer по отдельности, реализует ли он автоматически ReadWriter?
?
![[Что такое интерфейс#^iface-composition-example]]

Что такое Interface Guard? Как он выглядит синтаксически?
?
![[Что такое интерфейс#^iface-guard-def]]

Почему Interface Guard особенно полезен при реализации интерфейсов из сторонних библиотек?
?
![[Что такое интерфейс#^iface-guard-external]]

Что произойдёт при компиляции этого кода?
```go
type Shape interface { Area() float64 }
type Circle struct{ Radius float64 }
func (c Circle) Area() int { return 0 }
var _ Shape = (*Circle)(nil)
```
?
Компиляция упадёт с ошибкой: `Circle does not implement Shape (wrong type for Area)`. Interface Guard обнаружил несоответствие — возвращаемый тип `int` вместо `float64`.
![[Что такое интерфейс#^iface-guard-usecase]]

Когда использовать Interface Guard, а когда не нужно?
?
![[Что такое интерфейс#^iface-guard-usecase]]

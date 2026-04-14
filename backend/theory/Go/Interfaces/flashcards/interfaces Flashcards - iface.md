#flashcards/interfaces/iface

Сколько байт занимает iface и что в нём хранится?
?
![[iface структура#^iface-size]]

Что такое itab и для какой пары она уникальна?
?
![[iface структура#^iface-itab-unique]]

Для чего используется поле hash в itab?
?
![[iface структура#^iface-hash-typeswitch]]

Что такое поле fun в itab? Почему оно объявлено как [1]uintptr, хотя реально хранит N методов?
?
![[iface структура#^iface-fun-table]]

Опиши по шагам что происходит при вызове w.Write(buf) через интерфейс io.Writer
?
![[iface структура#^iface-dispatch-steps]]

Почему itab не вычисляется на этапе компиляции для всех возможных пар (интерфейс, тип)?
?
![[iface структура#^iface-no-early-binding]]

Как работает late binding itab: когда вычисляется и что потом?
?
![[iface структура#^iface-late-binding-cache]]

Что выведет этот код?
```go
var w io.Writer = os.Stdout
type MyWriter struct{}
func (m *MyWriter) Write(p []byte) (int, error) { return len(p), nil }
var w2 io.Writer = &MyWriter{}
fmt.Println(w == nil, w2 == nil)
```
?
`false false` — оба интерфейса имеют непустой tab (itab для *os.File и *MyWriter соответственно). Интерфейс nil только когда оба поля nil.
![[iface структура#^iface-size]]

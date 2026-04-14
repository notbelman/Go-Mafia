#flashcards/structs/defer_structs

Что выведет этот код?
```go
type Data struct{ Value string }
func (d Data) Print() { fmt.Println(d.Value) }

d := Data{Value: "original"}
defer d.Print()
d.Value = "changed"
```
?
`original` — value receiver копируется в момент `defer`. Последующее изменение `d.Value` не влияет на уже сохранённую копию.
![[Defer и структуры#^defer-value-receiver]]

Что выведет этот код?
```go
type Data struct{ Value string }
func (d *Data) Print() { fmt.Println(d.Value) }

d := &Data{Value: "original"}
defer d.Print()
d.Value = "changed"
```
?
`changed` — pointer receiver: копируется указатель, но он ссылается на тот же объект. Изменение `d.Value` видно при отложенном вызове.
![[Defer и структуры#^defer-pointer-receiver]]

Почему поведение defer отличается для value и pointer receiver? Объясни механизм.
?
![[Defer и структуры#^defer-receiver-why]]

Что выведет этот код? Разбери по шагам.
```go
func MakeData(ptr *int) Data {
    fmt.Println("make:", *ptr)
    return Data{Point: ptr}
}
func (d Data) Print(ptr *int) { fmt.Println("print:", *ptr) }

value := 1
ptr := &value
defer MakeData(ptr).Print(ptr)
value = 2
ptr = new(int)
MakeData(ptr)
```
?
```
make: 1   ← MakeData(ptr) сразу при defer (ptr→value=1)
make: 0   ← MakeData(ptr) в main (ptr→new int = 0)
print: 2  ← Print выполняется при выходе (старый ptr→value=2)
```
![[Defer и структуры#^defer-chain-detail]]

Сформулируй правило: что откладывается в `defer A().B().C()`?
?
![[Defer и структуры#^defer-chain-rule]]

Что скопируется при `defer A().B(ptr)` — и в какой момент?
?
![[Defer и структуры#^defer-chain-detail]]

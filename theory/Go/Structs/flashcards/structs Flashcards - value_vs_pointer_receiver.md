#flashcards/structs/value_vs_pointer_receiver

Что выведет этот код?
```go
type Account struct{ Balance int }
func (a Account) Add(amount int) { a.Balance += amount }
acc := Account{Balance: 100}
acc.Add(50)
fmt.Println(acc.Balance)
```
?
`100` — value receiver получает копию, изменения не видны снаружи.
![[Value vs Pointer receiver#^vr-copy-semantics]]

Что выведет этот код?
```go
type Account struct{ Balance int }
func (a *Account) Add(amount int) { a.Balance += amount }
acc := Account{Balance: 100}
acc.Add(50)
fmt.Println(acc.Balance)
```
?
`150` — pointer receiver меняет оригинал.
![[Value vs Pointer receiver#^pr-mutation]]

Что делает Go автоматически при вызове `acc.Add(50)` если `Add` — pointer-метод, а `acc` — переменная (не указатель)?
?
![[Value vs Pointer receiver#^vr-autosugar]]

Что выведет этот код?
```go
type Account struct{ Balance int }
type Client struct{ Acc *Account }
func (c Client) Add(amount int) { c.Acc.Balance += amount }
cl := Client{Acc: &Account{Balance: 0}}
cl.Add(100)
fmt.Println(cl.Acc.Balance)
```
?
`100` — value receiver скопировал `Client`, но копия указателя `Acc` указывает на тот же `Account`. Изменение видно.
![[Value vs Pointer receiver#^vr-indirect-ptr]]

Что выведет этот код?
```go
type Data struct{ Value int }
func (d Data) Get() int { return d.Value }
obj := Data{Value: 100}
fn := obj.Get
obj.Value = 999
fmt.Println(fn())
```
?
`100` — при `fn := obj.Get` зафиксировалась копия `obj` на тот момент. Последующие изменения не видны.
![[Value vs Pointer receiver#^vr-binding-value]]

Что выведет этот код?
```go
type Data struct{ Value int }
func (d *Data) Get() int { return d.Value }
obj := Data{Value: 100}
fn := obj.Get
obj.Value = 999
fmt.Println(fn())
```
?
`999` — при `fn := obj.Get` сохраняется указатель на `obj`. Изменения obj видны через fn.
![[Value vs Pointer receiver#^vr-binding-pointer]]

В каких двух случаях pointer receiver — обязателен?
?
![[Value vs Pointer receiver#^pr-must-use]]

Когда следует (но не обязательно) использовать pointer receiver?
?
![[Value vs Pointer receiver#^pr-should-use]]

Когда value receiver — обязателен?
?
![[Value vs Pointer receiver#^vr-must-use]]

Когда value receiver предпочтителен?
?
![[Value vs Pointer receiver#^vr-should-use]]

Какое правило по умолчанию при выборе типа ресивера?
?
![[Value vs Pointer receiver#^vr-default]]

Почему нельзя смешивать pointer и value ресиверы в одной структуре?
?
![[Value vs Pointer receiver#^vr-no-mix]]

- defer + **value receiver**: ресивер **копируется** при defer → изменения после defer не видны
- defer + **pointer receiver**: указатель копируется → изменения **видны**
- цепочка вызовов в defer: откладывается **только последний хвостик**, всё левее вычисляется сразу
- это следствие: defer откладывает функцию, аргументы (и ресивер) вычисляются сразу

---

## Value receiver — копия при defer

```go
type Data1 struct{ Value string }

func (d Data1) Print() { fmt.Println(d.Value) }  // value receiver

d1 := Data1{Value: "original"}
defer d1.Print()    // ресивер СКОПИРОВАН сейчас ("original")
d1.Value = "changed"
// При выходе напечатает: "original"
```

При `defer` с value receiver: ресивер копируется **в момент defer**, поэтому последующие изменения структуры не видны при отложенном вызове. ^defer-value-receiver

## Pointer receiver — указатель при defer

```go
type Data2 struct{ Value string }

func (d *Data2) Print() { fmt.Println(d.Value) }  // pointer receiver

d2 := &Data2{Value: "original"}
defer d2.Print()    // указатель скопирован (но указывает на тот же объект)
d2.Value = "changed"
// При выходе напечатает: "changed"
```

При `defer` с pointer receiver: копируется указатель, но он по-прежнему указывает на тот же объект — изменения **видны**. ^defer-pointer-receiver

**Почему?** defer откладывает вызов функции. Аргументы (включая ресивер) вычисляются сразу. Value receiver = копия объекта. Pointer receiver = копия указателя (тот же объект). ^defer-receiver-why

## Цепочка вызовов — хвостик

```go
func MakeData(ptr *int) Data {
    fmt.Println("make:", *ptr)
    return Data{Point: ptr}
}

func (d Data) Print(ptr *int) {
    fmt.Println("print:", *ptr)
}

value := 1
ptr := &value

defer MakeData(ptr).Print(ptr)  // что отложится?

value = 2
ptr = new(int)  // ptr теперь указывает на другое
MakeData(ptr)
```

**Порядок:**
1. `MakeData(ptr)` вызывается **сразу** (нужен ресивер для Print) → "make: 1"
2. `ptr` для Print **копируется сразу** (указывает на value)
3. `MakeData(ptr)` в main → "make: 0" (ptr уже другой)
4. При выходе: `Print(ptr)` → "print: 2" (ptr старый, но value=2)

**Правило:** в `defer A().B().C()` — откладывается только `C()`. Всё до `C()` вычисляется немедленно. ^defer-chain-rule

В цепочке `defer A().B()`: `A()` вызывается сразу (нужен ресивер для `B`), аргументы `B` копируются сразу, откладывается только сам вызов `B`. ^defer-chain-detail

## Связь
- [[Defer аргументы и ловушки]] — аргументы defer вычисляются сразу
- [[Value vs Pointer receiver]] — value vs pointer
- [[Методы и ресиверы]] — метод = функция + ресивер

- **pointer receiver**: метод должен мутировать объект, поле нельзя копировать (sync), объект большой
- **value receiver**: неизменяемость, базовые типы (int, string), косвенная ссылка через указатель внутри
- **не смешивать** типы ресиверов в одной структуре — почва для багов
- автосахар: Go берёт адрес / разыменовывает автоматически при вызове
- binding метода к объекту: value receiver → копия фиксируется, pointer → указатель сохраняется

---

## Value receiver — копия

```go
type Account struct{ Balance int }

func (a Account) Add(amount int) {
    a.Balance += amount  // меняем КОПИЮ
}

acc := Account{Balance: 100}
acc.Add(50)
fmt.Println(acc.Balance)  // 100 — не изменился!
```

Value receiver получает копию — изменения не видны снаружи. ^vr-copy-semantics

## Pointer receiver — мутация

```go
func (a *Account) Add(amount int) {
    a.Balance += amount  // меняем оригинал через указатель
}

acc := Account{Balance: 100}
acc.Add(50)
fmt.Println(acc.Balance)  // 150 ✅
```

Pointer receiver меняет оригинал. ^pr-mutation

## Автосахар Go

Go автоматически берёт адрес переменной при вызове pointer-метода (`acc.Add(50)` → `(&acc).Add(50)`) и разыменовывает при вызове value-метода (`ptr.Get()` → `(*ptr).Get()`). Работает только если переменная адресуема. ^vr-autosugar

## Косвенная ссылка — ловушка

```go
type Account struct{ Balance int }
type Client struct{ Acc *Account }  // указатель!

func (c Client) Add(amount int) {  // value receiver!
    c.Acc.Balance += amount         // но Balance ИЗМЕНИТСЯ
}

cl := Client{Acc: &Account{Balance: 0}}
cl.Add(100)
fmt.Println(cl.Acc.Balance)  // 100 ✅ изменился!
```

Value receiver скопировал `Client`, но внутри — **указатель** на `Account`. Копия указателя → тот же объект. Поэтому если изменяемые поля за указателем — value receiver допустим. ^vr-indirect-ptr

## Binding (привязка метода)

```go
type Data struct{ Value int }

func (d Data) Get() int    { return d.Value }   // value receiver
func (d *Data) PtrGet() int { return d.Value }  // pointer receiver

obj := Data{Value: 100}

fn1 := obj.Get      // value → КОПИЯ obj зафиксирована
fn2 := obj.PtrGet   // pointer → указатель на obj

obj.Value = 999

fn1()  // 100 — копия, изменение не видно
fn2()  // 999 — указатель, изменение видно
```

При привязке value-метода к переменной (`fn1 := obj.Get`) фиксируется копия на момент привязки. ^vr-binding-value

При привязке pointer-метода (`fn2 := obj.PtrGet`) сохраняется указатель — последующие изменения объекта видны. ^vr-binding-pointer

## Правила выбора (сводка)

**Указатель (должен):** мутация, некопируемые поля (`sync.Mutex`). ^pr-must-use

**Указатель (следует):** большой объект (бенчмаркать). ^pr-should-use

**Значение (должен):** гарантия неизменяемости. ^vr-must-use

**Значение (следует):** базовые типы, изменяемые поля за указателем внутри. ^vr-should-use

**По умолчанию:** если нет весомых причин — значение; при сомнениях — указатель. ^vr-default

## Не смешивать ресиверы

Смешивание pointer и value ресиверов в одной структуре — источник багов: разные методы будут видеть разное состояние объекта, интерфейсы могут не выполняться. ^vr-no-mix

## Связь
- [[Методы и ресиверы]] — метод = функция + ресивер
- [[Встраивание типов]] — ресиверы встроенных типов
- [[Defer и структуры]] — defer + value/pointer receiver

- вызов на **nil интерфейсе** `(nil, nil)` → **паника**: нет itab → негде искать метод
- вызов на интерфейсе с **nil значением** `(Type, nil)` → метод **вызовется** с nil receiver
- nil receiver + обращение к полям → **паника** внутри метода (разыменование nil)
- nil receiver + **без** обращения к полям → работает нормально
- идиома: **nil-safe методы** — проверка `if receiver == nil` в начале

---

## nil интерфейс → паника

```go
var w io.Writer          // (nil, nil)
w.Write([]byte("hi"))    // panic: nil pointer dereference
```

Вызов метода на nil интерфейсе `(nil, nil)` → **паника**: нет `itab` → нет таблицы методов → негде искать метод. ^nil-call-nil-iface-panic

## (Type, nil) → метод вызовется

```go
type MyWriter struct{}

func (w *MyWriter) Write(p []byte) (int, error) {
    if w == nil {
        return 0, errors.New("nil writer")
    }
    return len(p), nil
}

var w io.Writer = (*MyWriter)(nil)  // (*MyWriter, nil)
n, err := w.Write([]byte("hi"))    // вызов пройдёт!
// err = "nil writer"
```

```
w = (*MyWriter, nil)
+----------+--------+
| tab      | data   |
| *MyWriter| nil    |
+----------+--------+
     |
     v
   itab.fun[0] → (*MyWriter).Write  ← метод есть, вызываем с receiver=nil
```

Интерфейс `(Type, nil)` — `itab` есть → метод **найдётся и вызовется** с `receiver = nil`. ^nil-call-type-nil-calls

## nil receiver + обращение к полям → паника

```go
func (w *MyWriter) Write(p []byte) (int, error) {
    fmt.Println(w)         // <nil>
    fmt.Println(w == nil)  // true
    w.field = 1            // panic! разыменование nil
    return 0, nil
}
```

nil receiver можно использовать (например, проверить `w == nil`), но **обращение к полям** → паника: разыменование nil. ^nil-receiver-field-panic

## Сводка

| Ситуация | Результат |
|----------|-----------|
| `(nil, nil).Method()` | panic — нет itab |
| `(Type, nil).Method()` | вызов с nil receiver |
| nil receiver + поля | panic внутри метода |
| nil receiver + без полей | работает |

## Идиома: nil-safe методы

```go
func (u *User) Name() string {
    if u == nil {
        return ""
    }
    return u.name
}
```

**nil-safe метод**: проверка `if receiver == nil` в начале позволяет безопасно вызывать метод даже на nil receiver. ^nil-safe-method-idiom

## Особенность fmt.Println

```go
var err error = (*MyError)(nil)  // (Type, nil)
fmt.Println(err)                 // "<nil>" — не паника!
err.Error()                      // panic!
```

`fmt.Println` внутренне перехватывает панику (`catchPanic`) при вызове методов на nil-receiver. При **явном** вызове `.Error()` — паника не перехвачена. ^nil-fmtprintln-catchpanic

## Связь
- [[nil интерфейса]] — когда интерфейс == nil, а когда нет
- [[iface структура]] — itab нужен для вызова метода
- [[Методы и ресиверы]] — nil receiver = функция с nil первым аргументом

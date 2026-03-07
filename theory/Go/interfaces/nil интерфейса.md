- интерфейс == nil **только когда ОБА поля nil**: и type, и data
- присвоил типизированный nil указатель → type заполнен → интерфейс **НЕ nil**
- классическая ошибка: функция возвращает error, внутри `var err *MyError = nil`, `return err` → error != nil
- **правило**: если возвращаешь интерфейс и хочешь nil — возвращай **явный `nil`**, не типизированный указатель
- ловушка встречается не только при return, но и при **передаче аргументом** в функцию

---

## nil интерфейс — оба поля nil

```go
var w io.Writer          // w = (nil, nil)
fmt.Println(w == nil)    // true

var a any                // a = (nil, nil)
fmt.Println(a == nil)    // true
```

Интерфейс равен `nil` **только когда оба поля nil**: и `tab`/`_type`, и `data`. ^nil-iface-both-nil

## Типизированный nil → интерфейс НЕ nil

```go
var f *os.File = nil     // f = nil указатель
var w io.Writer = f      // w = (*os.File, nil) — тип ЕСТЬ
fmt.Println(w == nil)    // false!
```

```
w после присваивания nil указателя:
+----------+--------+
| tab      | data   |
| *os.File | nil    |  ← тип НЕ nil → интерфейс НЕ nil
+----------+--------+
```

Присваивание типизированного nil указателя в интерфейс → поле `tab`/`_type` заполнено → интерфейс **НЕ nil**, даже если `data == nil`. ^nil-typed-nil-not-nil

## Классическая ошибка с error

```go
func readFile(path string) error {
    var err *MyError = nil  // типизированный nil
    if path == "" {
        return err  // возвращаем (*MyError, nil) — НЕ nil!
    }
    // ...
    return nil
}

err := readFile("")
fmt.Println(err != nil)  // true! хотя значение внутри nil
```

Классическая ошибка Go: `var err *MyError = nil; return err` — возвращает `(*MyError, nil)`, а не `(nil, nil)`. Caller проверяет `err != nil` → **true**. ^nil-classic-error-bug

## Как правильно

```go
func readFile(path string) error {
    var err *MyError
    if path == "" {
        return nil  // ✅ явный nil → (nil, nil)
    }
    // ... если ошибка случилась:
    if err == nil {
        return nil  // ✅ проверили и вернули явный nil
    }
    return err
}
```

**Правило**: если функция возвращает интерфейс и нужен nil — возвращать **явный `nil`**, а не типизированный указатель. ^nil-return-explicit-nil

## Ловушка при передаче аргументом

```go
func check(v any) {
    fmt.Println(v == nil)
}

var d *Data = nil
check(d)  // false! — передали (*Data, nil), тип есть
```

Ловушка работает не только при `return`, но и при **любом** присваивании типизированного nil в интерфейс — в том числе при передаче аргументом. ^nil-trap-arg-passing

## Проверка значения внутри интерфейса

```go
// Крайний случай — если нужно проверить data == nil
if w != nil && !reflect.ValueOf(w).IsNil() {
    // теперь точно не nil
}
```

Проверить что `data == nil` у ненулевого интерфейса можно через `reflect.ValueOf(w).IsNil()`. ^nil-reflect-check

## Связь
- [[iface структура]] — два поля: tab + data
- [[eface]] — пустой интерфейс: _type + data (та же проблема)
- [[Вызов метода на nil]] — что происходит при вызове метода на (Type, nil)

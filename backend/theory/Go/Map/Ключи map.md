[[map Flashcards - keys]]
**Какие типы могут быть ключами:**

Любые comparable — те, что поддерживают ==: ^key-comparable-types
- bool, int, float, string
- pointer, channel
- array (если элементы comparable)
- struct (если все поля comparable)
- interface (может паниковать в runtime если внутри non-comparable)

**Не могут быть ключами:** slice, map, func — не поддерживают ==. ^key-forbidden-types
```go
[]int{1,2} == []int{1,2}  // ошибка компиляции
```

**Struct как ключ:**
```go
type Point struct { X, Y int }
m := map[Point]string{}
m[Point{1, 2}] = "ok"  // работает — все поля comparable

type Bad struct { Data []int }
m := map[Bad]string{}  // ошибка компиляции — slice внутри
```
^key-struct

**Pointer как ключ:**

Сравниваются адреса, не значения. ^key-pointer-semantics
```go
p1 := &Point{1, 2}
p2 := &Point{1, 2}
m[p1] = "first"
m[p2] = "second"  // другой ключ — разные адреса
len(m) == 2
```
^key-pointer-example

**Изменение объекта под pointer-ключом:**

Map не ломается — ключ это адрес, он не изменился. Но логика может сломаться: ^key-pointer-mutation
```go
p := &Point{1, 2}
m[p] = "point (1,2)"
p.X = 100  // изменили объект
// ключ тот же (адрес), но p теперь {100, 2}
```

**Interface как ключ — паника в runtime:**

Если в interface лежит non-comparable тип (например, slice), компилятор не поймает ошибку, но в runtime будет паника при попытке использовать такой ключ. ^key-interface-panic
```go
var k interface{} = []int{1, 2}
m := map[interface{}]string{}
m[k] = "val"  // panic: runtime error: hash of unhashable type []int
```

## Связь
- [[Хэш-таблицы теория]] — зачем ключи должны быть сравниваемыми (коллизии)
- [[Итерация и мутация]] — float как ключ: потеря точности
- [[операции чтения и вставки]] — сравнение ключей при поиске

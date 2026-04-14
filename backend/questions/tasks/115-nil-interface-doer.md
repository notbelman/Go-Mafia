---
type: task
companies:
  - Napoleon IT
topic: Go
subtopic:
  - Interface
  - Nil
  - Value Receiver
title: nil interface ловушка — будет ли ошибка
---

## Условие

Будет ли фатальная ошибка? В какую часть кода попадём и почему?

## Пример
```go
package main

import (
	"fmt"
	"log"
)

func main() {
	doer := NewDoer()
	if doer == nil {
		log.Fatalln("Doer object is nil, terminating...")
	}

	doer.do()
}

type Doer interface {
	do()
}

type Object struct{}

func NewDoer() Doer {
	var obj *Object
	return obj
}

func (o Object) do() {
	fmt.Println("Doer is doing something...")
}
```

## Решение

**Паника** — `runtime error: invalid memory address or nil pointer dereference`.

### Разбор по шагам

1. `NewDoer()` создаёт `var obj *Object` — nil указатель на Object
2. `return obj` — возвращаем nil `*Object` как интерфейс `Doer`
3. Под капотом интерфейс — два поля `(type, data)`. Тип заполнен (`*Object`), data = nil:
   ```
   doer = iface{tab: *Object, data: nil}
   ```
4. `doer == nil` → **false** — тип есть, значит интерфейс не nil. В `log.Fatalln` не попадаем.
5. `doer.do()` — вызов метода. Receiver — `(o Object)` — **value receiver**.
6. Runtime пытается разыменовать nil `*Object` чтобы получить значение `Object` для value receiver → **паника**.

### Ключевой момент

Если бы receiver был pointer (`func (o *Object) do()`), паники бы не было — метод на nil pointer receiver вызывается нормально, если не трогать поля. Но здесь value receiver — Go обязан скопировать значение, для этого нужно разыменовать указатель, а он nil.

### Фикс

```go
func NewDoer() Doer {
	var obj *Object
	if obj == nil {
		return nil // возвращаем nil интерфейс, не nil указатель в интерфейсе
	}
	return obj
}
```

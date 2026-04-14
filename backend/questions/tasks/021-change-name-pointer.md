---
type: task
companies:
  - AVITO
topic: Go
subtopic:
  - Pointers
  - Structs
title: changeName - изменение структуры через указатель
explained: "true"
---

## Условие
Что выведет программа? Как исправить, чтобы имя изменилось?

```go
package main

import "fmt"

type Person struct {
	Name string
}

// person — это КОПИЯ указателя, не сам указатель из main
// в main: person = 0xABC → {Name: "Bob"}
// в функции: person (копия) = 0xABC → {Name: "Bob"}
// оба указывают на один объект, но это ДВЕ разные переменные
func changeName(person *Person) {
	// создаём новый объект Person по адресу 0xDEF
	// и переназначаем ЛОКАЛЬНУЮ копию указателя на него
	// в функции: person = 0xDEF → {Name: "Alice"}
	// в main:    person = 0xABC → {Name: "Bob"} — не изменился!
	person = &Person{
		Name: "Alice",
	}
	// когда функция завершится, локальная копия указателя умрёт
	// объект {Name: "Alice"} по адресу 0xDEF станет мусором для GC
}

func main() {
	person := &Person{
		Name: "Bob",
	}
	fmt.Println(person.Name) // "Bob"

	// передаём КОПИЮ указателя в функцию
	// это как: funcPerson := person (оба = 0xABC)
	// функция может менять данные по адресу 0xABC (person.Name = "Alice")
	// но НЕ может изменить куда указывает person в main
	changeName(person)

	fmt.Println(person.Name) // "Bob" — указатель в main не изменился
}
```

**Ответ:** Выведет "Bob" оба раза. В `changeName` переназначается локальная копия указателя, исходный объект не меняется.

## Решение

**Способ 1: изменить поле напрямую**

```go
func changeName(person *Person) {
    person.Name = "Alice"
}
```

**Способ 2: двойной указатель**

```go
func changeName(person **Person) {
    *person = &Person{
        Name: "Alice",
    }
}

func main() {
    person := &Person{Name: "Bob"}
    changeName(&person)
    fmt.Println(person.Name) // "Alice"
}
```

**Способ 3: разыменование и присвоение**

```go
func changeName(person *Person) {
    *person = Person{
        Name: "Alice",
    }
}
```

**Объяснение:** При передаче указателя в функцию создаётся копия этого указателя. Переназначение `person = &Person{...}` меняет только локальную копию, а не исходный указатель в `main`. Чтобы изменить данные, нужно либо менять поля через разыменование, либо использовать двойной указатель.

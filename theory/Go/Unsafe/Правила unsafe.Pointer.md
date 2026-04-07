6 легальных паттернов из документации. Всё остальное — undefined behavior.

## 1. Каст *T → unsafe.Pointer → *U

Реинтерпретация памяти. Оба типа должны иметь совместимый memory layout. ^rule1-cast-types

```go
p := Point2D{X: 100, Y: 200}
v := *(*Vec2)(unsafe.Pointer(&p))  // ок если layout одинаковый
```

## 2. Pointer → uintptr (только для печати)

```go
fmt.Printf("addr: %x\n", uintptr(unsafe.Pointer(&x)))
```

Pointer → uintptr для печати — допустимо. Обратно uintptr → Pointer конвертировать **нельзя** (кроме паттернов 3-5). ^rule2-print-only

## 3. Арифметика — в одном выражении

Конверсия uintptr → Pointer **обязана** быть в одном выражении — иначе GC может сдвинуть объект и uintptr протухнет. ^rule3-one-expr

```go
// ✅ одно выражение
p := unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.age))

// ❌ сохранил в переменную — dangling
u := uintptr(unsafe.Pointer(&s))
p := unsafe.Pointer(u + offset)
```

С Go 1.17 есть `unsafe.Add` — делает то же самое безопаснее: ^rule3-unsafe-add

```go
p := unsafe.Add(unsafe.Pointer(&s), unsafe.Offsetof(s.age))
```

## 4. Syscall аргументы

Конверсия в uintptr допустима **только прямо в аргументе** вызова. Компилятор специально удерживает объект от GC до конца вызова. ^rule4-syscall

```go
syscall.Syscall(SYS_READ, uintptr(fd), uintptr(unsafe.Pointer(p)), uintptr(n))
```

## 5-6. reflect.Value и SliceHeader/StringHeader

`reflect.Value.Pointer()` возвращает uintptr — конвертировать обратно можно только сразу: ^rule5-reflect

```go
p := unsafe.Pointer(reflect.ValueOf(&x).Pointer()) // сразу в одном выражении
```

`SliceHeader/StringHeader` — deprecated с Go 1.20. ^rule6-sliceheader-deprecated

Вместо них: `unsafe.String`, `unsafe.StringData`, `unsafe.Slice`, `unsafe.SliceData`. ^rule6-new-funcs

## go vet

`go vet` проверяет паттерны unsafe.Pointer и ловит нарушения правил. Если `go vet` ругается — код невалиден. ^govet-check

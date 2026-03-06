- `unsafe.String` / `unsafe.Slice` — конверсия **без аллокации**, string и []byte шарят память ^unsafe-no-alloc
- результат: ×15 быстрее обычной конверсии, **0 аллокаций** ^unsafe-perf
- опасно: изменение []byte ломает string (map, горутины, оптимизации компилятора) ^unsafe-danger-summary
- deprecated способ: каст `*(*string)(unsafe.Pointer(&b))` — быстрее, но **layout может поменяться** ^unsafe-deprecated
- `strings.Builder.String()` внутри работает именно так ^unsafe-builder-internal

---

## Современный способ (Go 1.20+)

```go
// []byte → string без копирования
func bytesToString(b []byte) string {
    return unsafe.String(&b[0], len(b))
}

// string → []byte без копирования
func stringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}
```
^unsafe-modern-api

## Почему опасно

```go
b := []byte("hello")
s := bytesToString(b)  // s и b шарят память

b[0] = 'H'             // изменили b
fmt.Println(s)         // "Hello" — s тоже изменилась!
```
^unsafe-mutation-example

Нарушили иммутабельность. Последствия: map с таким ключом сломается (хеш изменился), строка в другой горутине неожиданно изменится, компилятор заоптимизировал что-то рассчитывая на иммутабельность. ^unsafe-consequences

## Когда можно использовать

1. `[]byte` больше никогда не изменится ^unsafe-safe-condition-1
2. Результат не сохраняется надолго (временная строка для lookup) ^unsafe-safe-condition-2
3. Профилировщик показал что конверсия — bottleneck ^unsafe-safe-condition-3

## Deprecated: каст указателя

```go
// []byte → string (старый способ)
s := *(*string)(unsafe.Pointer(&b))
```
^unsafe-deprecated-cast

Работает потому что у string и slice **два первых поля совпадают** (ptr, len). ^unsafe-why-works

Быстрее чем `unsafe.String` (нет вызовов функций, нет проверок). Но **layout может поменяться** в будущей версии Go — unsafe не защищён Go1 compatibility guarantee. Результат: undefined behavior, термоядерная бага без паники. ^unsafe-deprecated-risk

## Строковый литерал — нельзя менять даже через unsafe

```go
s := "hello"                          // text segment (read-only)
b := unsafe.Slice(unsafe.StringData(s), len(s))
b[0] = 'H'                           // 💥 SIGBUS — crash!
```
^unsafe-literal-sigbus

Строковые литералы живут в text segment (read-only память). ОС убивает процесс при попытке записи. ^unsafe-text-segment

## Связь
- [[byte конверсия]] — безопасная конверсия с копированием
- [[иммутабельность]] — что ломается при нарушении
- [[strings Builder]] — Builder.String() использует unsafe внутри
- [[interning]] — строковые литералы в text segment

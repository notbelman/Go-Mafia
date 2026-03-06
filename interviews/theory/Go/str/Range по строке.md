**for range итерирует по рунам, не байтам.** Индекс — это позиция байта. ^range-runes-not-bytes

| Способ                        | Что получаем  | Тип элемента |
| ----------------------------- | ------------- | ------------ |
| `for i, r := range s`         | руну (символ) | rune (int32) |
| `for i := 0; i < len(s); i++` | байт          | byte (uint8) |
| `for i, b := range []byte(s)` | байт          | byte (uint8) |

^range-ways-table

**Подсчёт символов:**
```go
len(s)                      // байты: 8
utf8.RuneCountInString(s)   // руны: 4
len([]rune(s))              // руны: 4 (но аллокация)
```
^range-count-methods

`len([]rune(s))` даёт количество рун, но создаёт аллокацию в отличие от `utf8.RuneCountInString(s)`. ^range-rune-alloc

```go
s := "hi世界"

for i, r := range s {
    fmt.Printf("%d: %c\n", i, r)
}
// 0: h
// 1: i
// 2: 世   ← индекс 2, но это байты 2,3,4
// 5: 界   ← индекс 5, не 3!
```
^range-index-is-byte-offset

**Обычный for — по байтам:**
```go
for i := 0; i < len(s); i++ {
    fmt.Printf("%d: %x\n", i, s[i])
}
// 0: 68        (h)
// 1: 69        (i)
// 2: e4        ← первый байт '世'
// 3: b8
// 4: 96
// 5: e7        ← первый байт '界'
// 6: 95
// 7: 8c
```
^range-plain-for-bytes

## Связь
- [[Rune и UTF-8]] — что такое руна
- [[Как Go понимает сколько байт в символе]] — маски для декодирования
- [[byte конверсия]] — range []byte(s) без аллокации (оптимизация компилятора)

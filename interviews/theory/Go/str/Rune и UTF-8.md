**rune** — это int32, представляет один Unicode code point (символ). ^rune-definition

**Строки в Go — это UTF-8 байты.** Один символ может занимать 1-4 байта: ^str-utf8-bytes

```go
s := "hello世界"

len(s)                        // 11 байт (5 + 3 + 3)
utf8.RuneCountInString(s)     // 7 символов

// ASCII: 1 байт = 1 символ
// 'h' = 1 байт (0x68)

// Китайский: 3 байта = 1 символ  
// '世' = 3 байта (0xe4 0xb8 0x96)
```
^rune-byte-sizes

**Индексация — по байтам, не символам:** ^str-indexing-bytes

```go
s := "привет"
s[0]           // 0xd0 — первый байт буквы 'п', не сама буква
string(s[0])   // мусор
```
^str-index-garbage

**Получить символ:**
```go
r := []rune(s)[0]    // 'п' (int32 = 1087)
string(r)            // "п"
```
^rune-get-char

**rune literal:**
```go
r := 'A'        // rune = 65
r2 := '世'      // rune = 19990
r3 := '\n'      // rune = 10
```
^rune-literals

## Связь
- [[Как Go понимает сколько байт в символе]] — битовые маски UTF-8
- [[Range по строке]] — range декодирует руны автоматически
- [[len и подстроки]] — len = байты, не символы
- [[Кодировки ASCII UTF-32]] — почему выбрали UTF-8

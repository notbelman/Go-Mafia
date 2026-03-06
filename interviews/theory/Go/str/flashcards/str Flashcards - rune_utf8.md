#flashcards/str/rune_utf8

Что такое `rune` в Go? Какой underlying тип?
?
![[Rune и UTF-8#^rune-definition]]

Строки в Go — это что? Сколько байт занимает один символ?
?
![[Rune и UTF-8#^str-utf8-bytes]]

Сколько байт занимает ASCII-символ и китайский иероглиф в UTF-8 строке Go?
?
![[Rune и UTF-8#^rune-byte-sizes]]

Что выведет этот код?
```go
s := "привет"
fmt.Printf("%x\n", s[0])
fmt.Println(string(s[0]))
```
?
`d0` и мусор (один байт кириллицы, не символ). Индексация строки в Go — по байтам. `s[0]` — это первый байт 'п' (0xd0), а не сама буква. `string(s[0])` конвертирует байт в строку, получается некорректный UTF-8.
![[Rune и UTF-8#^str-indexing-bytes]]
![[Rune и UTF-8#^str-index-garbage]]

Как правильно получить первый символ (руну) из строки?
?
![[Rune и UTF-8#^rune-get-char]]

Каким числом представлена руна `'世'`? А `'A'`? А `'\n'`?
?
![[Rune и UTF-8#^rune-literals]]

Что выведет?
```go
s := "hello世界"
fmt.Println(len(s))
fmt.Println(utf8.RuneCountInString(s))
```
?
```
11
7
```
hello = 5 байт (ASCII), 世 = 3 байта, 界 = 3 байта → итого 11 байт. Символов 7.
![[Rune и UTF-8#^rune-byte-sizes]]

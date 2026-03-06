#flashcards/str/range_string

Что итерирует `for range` по строке — байты или руны? Какой тип у индекса?
?
![[Range по строке#^range-runes-not-bytes]]

Три способа итерироваться по строке в Go. Что получаем и какой тип у элемента в каждом случае?
?
![[Range по строке#^range-ways-table]]

Почему индекс в `for i, r := range s` прыгает не по 1 для UTF-8 строки?
?
![[Range по строке#^range-index-is-byte-offset]]

Три способа получить количество символов (рун) в строке. В чём разница между `utf8.RuneCountInString(s)` и `len([]rune(s))`?
?
![[Range по строке#^range-count-methods]]
![[Range по строке#^range-rune-alloc]]

Что выведет этот код?
```go
s := "hi世界"
for i, r := range s {
    fmt.Printf("%d: %c\n", i, r)
}
```
?
```
0: h
1: i
2: 世
5: 界
```
Индекс — это байтовое смещение. '世' занимает байты 2,3,4, поэтому '界' начинается с байта 5, а не 3.
![[Range по строке#^range-index-is-byte-offset]]

Что выведет этот код?
```go
s := "hi世界"
for i := 0; i < len(s); i++ {
    fmt.Printf("%d: %x\n", i, s[i])
}
```
?
```
0: 68   (h)
1: 69   (i)
2: e4   (первый байт '世')
3: b8
4: 96
5: e7   (первый байт '界')
6: 95
7: 8c
```
Обычный for по строке идёт по байтам. Многобайтовые символы разбиваются на отдельные байты.
![[Range по строке#^range-plain-for-bytes]]

Что выведет?
```go
s := "hi世界"
fmt.Println(len(s))
fmt.Println(utf8.RuneCountInString(s))
```
?
```
8
4
```
`len` возвращает байты: h(1)+i(1)+世(3)+界(3)=8. `RuneCountInString` — символы: 4.
![[Range по строке#^range-count-methods]]

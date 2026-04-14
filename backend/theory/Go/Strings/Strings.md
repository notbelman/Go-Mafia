## Strings в Go

### Основы

#### [[структура]]
string = (ptr, len), immutable, не null-terminated

#### [[Виды строк]]
string literal, raw string (backtick), string vs []byte vs []rune

#### [[иммутабельность]]
Почему string иммутабельна, безопасное sharing без копирования

#### [[len и подстроки]]
len() возвращает байты, не символы; подстрока делит backing array

---

### Unicode и кодировки

#### [[Rune и UTF-8]]
rune = int32 = Unicode code point, UTF-8 переменная ширина

#### [[Как Go понимает сколько байт в символе]]
Первый байт определяет длину (1-4 байта), invalid byte sequences

#### [[Кодировки ASCII UTF-32]]
ASCII подмножество UTF-8, UTF-16/UTF-32 — почему Go выбрал UTF-8

#### [[Range по строке]]
range итерирует по rune (не байтам), индекс = байтовая позиция

---

### Конвертации

#### [[byte конверсия]]
string → []byte (копия), []byte → string (копия), когда компилятор оптимизирует

#### [[Unsafe string to byte]]
unsafe конвертация без копирования, когда безопасно

#### [[interning]]
String interning, constants дедуплицируются компилятором

---

### Оптимизация

#### [[strings Builder]]
strings.Builder для конкатенации, WriteString, Reset, Grow

#### [[Строки утечки памяти]]
Подстрока держит весь backing array — string([]byte(s[:])) для копии

#### [[Сравнение строк]]
== байтовое сравнение, strings.EqualFold для case-insensitive

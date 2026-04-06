- два среза шарят один array → append может **перезаписать чужие данные**
- `cap > len` у подсреза → append пишет в общий array, не в новый
- дескрипторы **независимы**: изменение len/cap одного НЕ влияет на другой
- при реаллокации подсреза новый cap считается от **его len**, не от cap оригинала
- решение: `a[:2:2]` (full slice) или `copy` в новый slice

---
[[slice Flashcards - subslice_bugs]]
## Баг #1: append перезаписывает чужие данные

```go
a := []int{1, 2, 3, 4, 5}
b := a[:2]               // b = [1 2], cap = 5

b = append(b, 99)        // cap хватает → пишет в общий array
fmt.Println(a)           // [1 2 99 4 5] ← a[2] перезаписан!
```

^slice-bug-overwrite

Условие бага: `cap(b) > len(b)` → append не реаллоцирует, а пишет в следующую позицию общего array, перезаписывая данные оригинала. ^slice-bug-overwrite-condition

## Баг #2: непредсказуемое расхождение

```go
data := make([]int, 4, 6)  // len=4, cap=6
sub := data[2:6]            // len=4, cap=4

sub = append(sub, 1)        // cap=4, некуда → реаллокация!
// sub теперь на НОВОМ array, data — на старом
// Изменения в sub больше не видны в data
```

^slice-bug-diverge

При реаллокации подсреза новый `cap` считается от длины подсреза (`len(sub) = 4`), а не от cap оригинала (`cap(data) = 6`). Новый cap = 8. ^slice-bug-diverge-newcap

## Баг #3: дескрипторы независимы

```go
data := make([]int, 3, 6)
sub := data[1:3]

sub = append(sub, 77)  // len(sub) выросла, len(data) — нет
fmt.Println(len(data))  // 3 — не изменилось!
// Но data[3] = 77, просто data об этом "не знает"
```

^slice-bug-independent-descriptors

Каждый срез — свой дескриптор `{ptr, len, cap}`. Изменение `len`/`cap` одного **никак не влияет** на другой, даже если шарят array. Данные записаны в array, но оригинал не "знает" о новом элементе, потому что его `len` не изменился. ^slice-bug-independent-why

## Решение: full slice expression

```go
b := a[:2:2]  // cap = 2, ограничен!
b = append(b, 99)  // cap=2 → реаллокация → новый array
fmt.Println(a)     // [1 2 3 4 5] ← не тронут
```

^slice-fix-full-slice

`a[:2:2]` — синтаксис `[low:high:max]`, где третий аргумент ограничивает `cap = max - low`. После этого append гарантированно реаллоцирует. ^slice-fix-full-slice-syntax

## Решение: копирование

```go
sub := make([]int, 2)
copy(sub, original[1:3])  // независимый array
```

^slice-fix-copy

## Связь
- [[Full slice expression]] — ограничение cap третьим аргументом
- [[Subslicing нарезка]] — механика получения подсрезов
- [[Slice утечки памяти]] — когда маленький подсрез держит большой array
- [[Slice copy]] — правильное копирование

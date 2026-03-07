- `s[i:j]` — полуинтервал: `i` включён, `j` нет (`i` и `j` — это индексы underlying array). Новый дескриптор, **тот же array** ^sub-same-array
- `len = j - i`, `cap = cap(базового) - i` — от начала нарезки до конца cap базового ^sub-cap-rule
- длина производного может быть **> длины** базового, cap — **никогда** ^sub-len-vs-cap
- nil slice: `s[0:0]`, `s[:0]` — **валидны**, append к nil — ок ^sub-nil-valid
- ресайз: `s = s[:cap(s)]` — расширить len до cap через нарезку ^sub-resize

---
[[slice Flashcards - subslicing]]
## Базовый синтаксис

```go
s := []int{0, 1, 2, 3, 4}
s[1:3]  // [1 2]       — с 1-го до 3-го (не включая)
s[2:]   // [2 3 4]     — с 2-го до конца
s[:3]   // [0 1 2]     — с начала до 3-го
s[:]    // [0 1 2 3 4] — копия дескриптора
```

^sub-syntax

Отрицательные индексы (как в Python) — нельзя. ^sub-no-negative

## Правила len и cap

```go
s := make([]int, 4, 6)  // len=4, cap=6
sub := s[2:4]           // len=2, cap=4 (до конца базового cap)
```

^sub-len-cap-example

Ключевое правило: длина производного может быть > длины базового, но cap — никогда. ^sub-len-gt-cap-never

## Ресайз через нарезку

```go
s := make([]int, 3, 6)
s[3] = 42       // ❌ panic — за пределами len
s = s[:cap(s)]  // расширили len до cap
s[3] = 42       // ✅ теперь ок
```
^sub-resize-example

## Операции с nil slice

```go
var s []int          // nil
_ = s[0:0]           // ✅ валидно, пустой slice
_ = s[:0]            // ✅ валидно
append(s, 1)         // ✅ работает
for range s { ... }  // ✅ 0 итераций
```

^sub-nil-ops

## Шаринг буфера

```go
data := []int{1, 2, 3, 4, 5, 0}
sub := data[1:3]  // [2 3], cap=5, общий array

sub[0] = 999
fmt.Println(data)  // [1 999 3 4 5 0] — изменилось!

sub = append(sub, 77)  // len(sub) растёт, len(data) — нет
// data[3] теперь = 77 (перезаписали!)
```

^sub-sharing-example

Запись через подсрез модифицирует общий underlying array. `append` в `sub` при наличии cap записывает данные в позиции за `len(data)` — `data` этого не видит, но данные в её буфере изменились. ^sub-sharing-why

## Связь
- [[Full slice expression]] — третий аргумент для ограничения cap
- [[Slice от slice баги]] — баги при шаринге буфера
- [[Удаление и очистка slice]] — нарезка для удаления элементов

- копируется **header** `{ptr, len, cap}`, ptr тот же → элементы видны, len — нет ^app-header-copy
- `cap == len` → новый array → снаружи **ничего не видно** ^app-cap-eq-len
- `cap > len` → тот же array → данные записались, но **len снаружи старый** ^app-cap-gt-len
- опасный кейс: `copy := append(data, 5)` при `cap > len` — пишет в чужой array ^app-dangerous-case

---
[[slice Flashcards - append_in_func]]

Слайс = header `{ptr, len, cap}` — передаётся копия header, но `ptr` тот же. ^app-slice-header

`append` смотрит: есть место в `cap`?
  `cap == len` → новый array   → снаружи не видно
  `cap > len`  → тот же array  → данные видны, но `len` старый

^app-append-logic

**Пример 1: `cap == len` (новый array)**
```go
func add(s []int) {
    s = append(s, 4)  // новый array, main не узнает
}
s := []int{1, 2, 3}   // len=3, cap=3
add(s)
fmt.Println(s)        // [1 2 3] ← без изменений
```

^app-example-new-array

**Пример 2: `cap > len` (тот же array)**
```go
func add(s []int) {
    s = append(s, 4)  // тот же array
    s[0] = 999        // меняет общие данные
}
s := make([]int, 3, 10)  // len=3, cap=10
s[0], s[1], s[2] = 1, 2, 3
add(s)
fmt.Println(s)        // [999 2 3] ← s[0] изменился, но len=3
```

Четвёрка записалась в array, но main's `len` не знает об этом — он всё ещё 3. `fmt.Println(s)` печатает только `s[0:len]`. ^app-example-same-array

Хочешь изменения снаружи? Возвращай слайс или передавай `*[]int` ^app-fix

## Связь
- [[Как сделать append видимым]] — два способа решения
- [[Slice передача в функцию]] — что именно копируется
- [[append]] — почему append возвращает slice
